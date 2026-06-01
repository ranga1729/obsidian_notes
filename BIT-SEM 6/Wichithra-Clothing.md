Looking at your current schema and the project proposal, here are the missing tables needed for a production-grade clothing e-commerce system:

---

## 1. Authentication Tokens

Missing for email verification, password reset, phone OTP.

```prisma
model VerificationToken {
  id        String    @id @default(uuid()) @db.Uuid
  userId    String    @db.Uuid
  token     String    @unique @db.VarChar(255)
  type      TokenType
  expiresAt DateTime  @db.Timestamptz(6)
  usedAt    DateTime? @db.Timestamptz(6)
  createdAt DateTime  @default(now()) @db.Timestamptz(6)

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([token])
  @@index([userId])
  @@map("verification_tokens")
}

enum TokenType {
  EMAIL_VERIFICATION
  PASSWORD_RESET
  PHONE_VERIFICATION
}
```

---

## 2. Cart & CartItem

Your proposal requires persistent cart (even for guests via `sessionId`).

```prisma
model Cart {
  id        String    @id @default(uuid()) @db.Uuid
  userId    String?   @db.Uuid       // null for guest carts
  sessionId String?   @db.VarChar(255) // for guest carts
  expiresAt DateTime? @db.Timestamptz(6)
  createdAt DateTime  @default(now()) @db.Timestamptz(6)
  updatedAt DateTime  @updatedAt @db.Timestamptz(6)

  user      User?      @relation(fields: [userId], references: [id], onDelete: Cascade)
  cartItems CartItem[]

  @@index([userId])
  @@index([sessionId])
  @@map("carts")
}

model CartItem {
  id          String   @id @default(uuid()) @db.Uuid
  cartId      String   @db.Uuid
  inventoryId String   @db.Uuid  // ties to exact product+size+color combo
  quantity    Int      @default(1)
  addedAt     DateTime @default(now()) @db.Timestamptz(6)
  updatedAt   DateTime @updatedAt @db.Timestamptz(6)

  cart      Cart      @relation(fields: [cartId], references: [id], onDelete: Cascade)
  inventory Inventory @relation(fields: [inventoryId], references: [id])

  @@unique([cartId, inventoryId])
  @@map("cart_items")
}
```

---

## 3. Order, OrderItem, OrderAddress, OrderStatusHistory

Critical for the checkout flow. Note `OrderAddress` **snapshots** the address so it's immutable even if the user later edits their address.

```prisma
model Order {
  id              String        @id @default(uuid()) @db.Uuid
  orderNumber     String        @unique @db.VarChar(50) // e.g. WC-20260001
  userId          String        @db.Uuid
  status          OrderStatus   @default(PENDING)
  paymentStatus   PaymentStatus @default(PENDING)
  couponId        String?       @db.Uuid
  subtotal        Decimal       @db.Decimal(10, 2)
  discountAmount  Decimal       @default(0) @db.Decimal(10, 2)
  shippingFee     Decimal       @default(0) @db.Decimal(10, 2)
  taxAmount       Decimal       @default(0) @db.Decimal(10, 2)
  totalAmount     Decimal       @db.Decimal(10, 2)
  notes           String?       @db.Text
  cancelReason    String?       @db.VarChar(255)
  createdAt       DateTime      @default(now()) @db.Timestamptz(6)
  updatedAt       DateTime      @updatedAt @db.Timestamptz(6)

  user                User                 @relation(fields: [userId], references: [id])
  coupon              Coupon?              @relation(fields: [couponId], references: [id])
  orderItems          OrderItem[]
  orderAddresses      OrderAddress[]
  orderStatusHistory  OrderStatusHistory[]
  payments            Payment[]
  shipments           Shipment[]
  returnRequests      ReturnRequest[]

  @@index([userId])
  @@index([status])
  @@index([paymentStatus])
  @@map("orders")
}

model OrderItem {
  id              String   @id @default(uuid()) @db.Uuid
  orderId         String   @db.Uuid
  inventoryId     String   @db.Uuid
  // Snapshot fields — preserve state at purchase time
  productName     String   @db.VarChar(200)
  sizeName        String   @db.VarChar(10)
  colorName       String   @db.VarChar(50)
  imageUrl        String?  @db.VarChar(500)
  quantity        Int
  unitPrice       Decimal  @db.Decimal(10, 2)
  discountAmount  Decimal  @default(0) @db.Decimal(10, 2)
  totalPrice      Decimal  @db.Decimal(10, 2)
  createdAt       DateTime @default(now()) @db.Timestamptz(6)

  order     Order     @relation(fields: [orderId], references: [id], onDelete: Cascade)
  inventory Inventory @relation(fields: [inventoryId], references: [id])
  returnItems ReturnItem[]

  @@index([orderId])
  @@map("order_items")
}

// Immutable snapshot of address at time of order
model OrderAddress {
  id           String      @id @default(uuid()) @db.Uuid
  orderId      String      @db.Uuid
  type         AddressType @default(DELIVERY)
  houseNo      String      @db.VarChar(20)
  addressLine1 String      @db.VarChar(255)
  addressLine2 String?     @db.VarChar(255)
  city         String      @db.VarChar(100)
  province     String      @db.VarChar(100)
  zipcode      String      @db.VarChar(20)
  country      String      @default("Sri Lanka") @db.VarChar(100)
  createdAt    DateTime    @default(now()) @db.Timestamptz(6)

  order Order @relation(fields: [orderId], references: [id], onDelete: Cascade)

  @@index([orderId])
  @@map("order_addresses")
}

model OrderStatusHistory {
  id        String      @id @default(uuid()) @db.Uuid
  orderId   String      @db.Uuid
  status    OrderStatus
  notes     String?     @db.VarChar(500)
  createdBy String?     @db.Uuid  // admin userId
  createdAt DateTime    @default(now()) @db.Timestamptz(6)

  order Order @relation(fields: [orderId], references: [id], onDelete: Cascade)

  @@index([orderId])
  @@map("order_status_history")
}

enum OrderStatus {
  PENDING
  CONFIRMED
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
  RETURNED
  REFUNDED

  @@map("order_status")
}

enum PaymentStatus {
  PENDING
  COMPLETED
  FAILED
  REFUNDED
  PARTIALLY_REFUNDED

  @@map("payment_status")
}
```

---

## 4. Payment

Ties to a payment gateway (e.g., PayHere for Sri Lanka).

```prisma
model Payment {
  id                    String        @id @default(uuid()) @db.Uuid
  orderId               String        @db.Uuid
  amount                Decimal       @db.Decimal(10, 2)
  currency              String        @default("LKR") @db.VarChar(10)
  method                PaymentMethod
  status                PaymentStatus @default(PENDING)
  gatewayProvider       String?       @db.VarChar(50)  // "payhere", "stripe"
  gatewayTransactionId  String?       @unique @db.VarChar(255)
  gatewayResponse       Json?         // full raw response for debugging
  failureReason         String?       @db.VarChar(255)
  paidAt                DateTime?     @db.Timestamptz(6)
  refundedAt            DateTime?     @db.Timestamptz(6)
  createdAt             DateTime      @default(now()) @db.Timestamptz(6)
  updatedAt             DateTime      @updatedAt @db.Timestamptz(6)

  order   Order    @relation(fields: [orderId], references: [id])
  refunds Refund[]

  @@index([orderId])
  @@map("payments")
}

enum PaymentMethod {
  CARD
  BANK_TRANSFER
  CASH_ON_DELIVERY
  PAYHERE
  OTHER

  @@map("payment_method")
}
```

---

## 5. Shipping & Shipment

Shipping rates by zone + tracking per order.

```prisma
model ShippingZone {
  id        String   @id @default(uuid()) @db.Uuid
  name      String   @unique @db.VarChar(100)
  provinces Json     // array of province names covered
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now()) @db.Timestamptz(6)
  updatedAt DateTime @updatedAt @db.Timestamptz(6)

  shippingRates ShippingRate[]

  @@map("shipping_zones")
}

model ShippingRate {
  id               String       @id @default(uuid()) @db.Uuid
  shippingZoneId   String       @db.Uuid
  name             String       @db.VarChar(100)  // "Standard", "Express"
  minOrderAmount   Decimal?     @db.Decimal(10, 2)
  price            Decimal      @db.Decimal(10, 2)
  estimatedDaysMin Int          @default(1)
  estimatedDaysMax Int          @default(5)
  isActive         Boolean      @default(true)
  createdAt        DateTime     @default(now()) @db.Timestamptz(6)
  updatedAt        DateTime     @updatedAt @db.Timestamptz(6)

  shippingZone ShippingZone @relation(fields: [shippingZoneId], references: [id])

  @@map("shipping_rates")
}

model Shipment {
  id               String         @id @default(uuid()) @db.Uuid
  orderId          String         @db.Uuid
  trackingNumber   String?        @db.VarChar(100)
  carrier          String?        @db.VarChar(100)  // "DHL", "Lanka Courier"
  status           ShipmentStatus @default(PROCESSING)
  estimatedDelivery DateTime?     @db.Timestamptz(6)
  shippedAt        DateTime?      @db.Timestamptz(6)
  deliveredAt      DateTime?      @db.Timestamptz(6)
  createdAt        DateTime       @default(now()) @db.Timestamptz(6)
  updatedAt        DateTime       @updatedAt @db.Timestamptz(6)

  order Order @relation(fields: [orderId], references: [id])

  @@index([orderId])
  @@map("shipments")
}

enum ShipmentStatus {
  PROCESSING
  SHIPPED
  IN_TRANSIT
  OUT_FOR_DELIVERY
  DELIVERED
  FAILED

  @@map("shipment_status")
}
```

---

## 6. Returns & Refunds

```prisma
model ReturnRequest {
  id        String       @id @default(uuid()) @db.Uuid
  orderId   String       @db.Uuid
  userId    String       @db.Uuid
  reason    String       @db.VarChar(500)
  status    ReturnStatus @default(REQUESTED)
  notes     String?      @db.Text
  createdAt DateTime     @default(now()) @db.Timestamptz(6)
  updatedAt DateTime     @updatedAt @db.Timestamptz(6)

  order       Order        @relation(fields: [orderId], references: [id])
  user        User         @relation(fields: [userId], references: [id])
  returnItems ReturnItem[]
  refund      Refund?

  @@index([orderId])
  @@map("return_requests")
}

model ReturnItem {
  id              String    @id @default(uuid()) @db.Uuid
  returnRequestId String    @db.Uuid
  orderItemId     String    @db.Uuid
  quantity        Int
  reason          String?   @db.VarChar(255)
  condition       String?   @db.VarChar(100) // "unopened", "damaged"
  createdAt       DateTime  @default(now()) @db.Timestamptz(6)

  returnRequest ReturnRequest @relation(fields: [returnRequestId], references: [id], onDelete: Cascade)
  orderItem     OrderItem     @relation(fields: [orderItemId], references: [id])

  @@map("return_items")
}

model Refund {
  id              String       @id @default(uuid()) @db.Uuid
  orderId         String       @db.Uuid
  paymentId       String       @db.Uuid
  returnRequestId String?      @unique @db.Uuid
  amount          Decimal      @db.Decimal(10, 2)
  reason          String?      @db.VarChar(255)
  status          RefundStatus @default(PENDING)
  processedAt     DateTime?    @db.Timestamptz(6)
  createdAt       DateTime     @default(now()) @db.Timestamptz(6)
  updatedAt       DateTime     @updatedAt @db.Timestamptz(6)

  order         Order          @relation(fields: [orderId], references: [id])
  payment       Payment        @relation(fields: [paymentId], references: [id])
  returnRequest ReturnRequest? @relation(fields: [returnRequestId], references: [id])

  @@map("refunds")
}

enum ReturnStatus {
  REQUESTED
  APPROVED
  REJECTED
  ITEM_RECEIVED
  REFUNDED

  @@map("return_status")
}

enum RefundStatus {
  PENDING
  PROCESSING
  COMPLETED
  FAILED

  @@map("refund_status")
}
```

---

## 7. Coupons & Promotions

```prisma
model Coupon {
  id                String     @id @default(uuid()) @db.Uuid
  code              String     @unique @db.VarChar(50)
  name              String     @db.VarChar(100)
  description       String?    @db.VarChar(255)
  type              CouponType
  value             Decimal    @db.Decimal(10, 2)  // % or fixed LKR
  minOrderAmount    Decimal?   @db.Decimal(10, 2)
  maxDiscountAmount Decimal?   @db.Decimal(10, 2)  // cap for percentage coupons
  usageLimit        Int?       // null = unlimited
  usedCount         Int        @default(0)
  perUserLimit      Int        @default(1)
  isActive          Boolean    @default(true)
  startsAt          DateTime?  @db.Timestamptz(6)
  expiresAt         DateTime?  @db.Timestamptz(6)
  createdAt         DateTime   @default(now()) @db.Timestamptz(6)
  updatedAt         DateTime   @updatedAt @db.Timestamptz(6)

  orders       Order[]
  couponUsages CouponUsage[]

  @@map("coupons")
}

model CouponUsage {
  id             String   @id @default(uuid()) @db.Uuid
  couponId       String   @db.Uuid
  userId         String   @db.Uuid
  orderId        String   @db.Uuid
  discountAmount Decimal  @db.Decimal(10, 2)
  usedAt         DateTime @default(now()) @db.Timestamptz(6)

  coupon Coupon @relation(fields: [couponId], references: [id])
  user   User   @relation(fields: [userId], references: [id])

  @@unique([couponId, orderId])
  @@map("coupon_usages")
}

enum CouponType {
  PERCENTAGE
  FIXED_AMOUNT
  FREE_SHIPPING

  @@map("coupon_type")
}
```

---

## 8. Wishlist

```prisma
model Wishlist {
  id        String   @id @default(uuid()) @db.Uuid
  userId    String   @unique @db.Uuid
  createdAt DateTime @default(now()) @db.Timestamptz(6)

  user          User           @relation(fields: [userId], references: [id], onDelete: Cascade)
  wishlistItems WishlistItem[]

  @@map("wishlists")
}

model WishlistItem {
  id         String   @id @default(uuid()) @db.Uuid
  wishlistId String   @db.Uuid
  productId  String   @db.Uuid
  addedAt    DateTime @default(now()) @db.Timestamptz(6)

  wishlist Wishlist @relation(fields: [wishlistId], references: [id], onDelete: Cascade)
  product  Product  @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@unique([wishlistId, productId])
  @@map("wishlist_items")
}
```

---

## 9. Product Reviews

```prisma
model ProductReview {
  id                String   @id @default(uuid()) @db.Uuid
  productId         String   @db.Uuid
  userId            String   @db.Uuid
  rating            Int      // 1–5
  title             String?  @db.VarChar(200)
  body              String?  @db.Text
  isVerifiedPurchase Boolean @default(false)
  isApproved        Boolean  @default(false)
  helpfulCount      Int      @default(0)
  createdAt         DateTime @default(now()) @db.Timestamptz(6)
  updatedAt         DateTime @updatedAt @db.Timestamptz(6)

  product      Product       @relation(fields: [productId], references: [id], onDelete: Cascade)
  user         User          @relation(fields: [userId], references: [id])
  reviewImages ReviewImage[]

  @@unique([productId, userId])  // one review per user per product
  @@index([productId])
  @@index([isApproved])
  @@map("product_reviews")
}

model ReviewImage {
  id        String   @id @default(uuid()) @db.Uuid
  reviewId  String   @db.Uuid
  imageUrl  String   @db.VarChar(500)
  sortOrder Int      @default(0)
  createdAt DateTime @default(now()) @db.Timestamptz(6)

  review ProductReview @relation(fields: [reviewId], references: [id], onDelete: Cascade)

  @@map("review_images")
}
```

---

## 10. Tags (Product Discoverability)

Useful for search and filtering beyond categories/designs.

```prisma
model Tag {
  id        String   @id @default(uuid()) @db.Uuid
  name      String   @unique @db.VarChar(50)
  slug      String   @unique @db.VarChar(50)
  createdAt DateTime @default(now()) @db.Timestamptz(6)

  productTags ProductTag[]

  @@map("tags")
}

model ProductTag {
  id        String   @id @default(uuid()) @db.Uuid
  productId String   @db.Uuid
  tagId     String   @db.Uuid
  createdAt DateTime @default(now()) @db.Timestamptz(6)

  product Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  tag     Tag     @relation(fields: [tagId], references: [id], onDelete: Cascade)

  @@unique([productId, tagId])
  @@map("product_tags")
}
```

---

## 11. Notifications

For order updates, low stock, etc.

```prisma
model Notification {
  id             String           @id @default(uuid()) @db.Uuid
  userId         String           @db.Uuid
  type           NotificationType
  title          String           @db.VarChar(200)
  message        String           @db.Text
  isRead         Boolean          @default(false)
  readAt         DateTime?        @db.Timestamptz(6)
  referenceId    String?          @db.Uuid    // orderId, productId, etc.
  referenceType  String?          @db.VarChar(50) // "ORDER", "PRODUCT"
  createdAt      DateTime         @default(now()) @db.Timestamptz(6)

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId, isRead])
  @@map("notifications")
}

enum NotificationType {
  ORDER_PLACED
  ORDER_CONFIRMED
  ORDER_SHIPPED
  ORDER_DELIVERED
  ORDER_CANCELLED
  PAYMENT_RECEIVED
  PAYMENT_FAILED
  RETURN_APPROVED
  RETURN_REJECTED
  REFUND_PROCESSED
  LOW_STOCK_ALERT      // for admin
  REVIEW_APPROVED

  @@map("notification_type")
}
```

---

## 12. Audit Log (Admin Actions Traceability)

```prisma
model AuditLog {
  id            String   @id @default(uuid()) @db.Uuid
  userId        String?  @db.Uuid
  action        String   @db.VarChar(100) // "CREATE", "UPDATE", "DELETE", "LOGIN"
  entity        String   @db.VarChar(100) // "Product", "Order", "User"
  entityId      String?  @db.Uuid
  oldValues     Json?
  newValues     Json?
  ipAddress     String?  @db.VarChar(45)  // IPv6 max length
  userAgent     String?  @db.VarChar(500)
  createdAt     DateTime @default(now()) @db.Timestamptz(6)

  @@index([userId])
  @@index([entity, entityId])
  @@map("audit_logs")
}
```

---

## 13. Site Configuration

For admin-configurable settings without code deploys.

```prisma
model SiteSetting {
  id          String   @id @default(uuid()) @db.Uuid
  key         String   @unique @db.VarChar(100)
  value       String   @db.Text
  description String?  @db.VarChar(255)
  updatedAt   DateTime @updatedAt @db.Timestamptz(6)

  @@map("site_settings")
}
```

---

## Summary

| Priority | Tables |
|---|---|
| **Must-have** | `Cart`, `CartItem`, `Order`, `OrderItem`, `OrderAddress`, `OrderStatusHistory`, `Payment`, `VerificationToken` |
| **High** | `Shipment`, `ShippingZone`, `ShippingRate`, `Coupon`, `CouponUsage`, `Notification` |
| **Medium** | `ReturnRequest`, `ReturnItem`, `Refund`, `ProductReview`, `ReviewImage`, `Wishlist`, `WishlistItem` |
| **Nice-to-have** | `Tag`, `ProductTag`, `AuditLog`, `SiteSetting` |

Key design decisions to note:
- **`OrderAddress`** snapshots the address at purchase time — never reference `Address` directly from `Order`
- **`OrderItem`** snapshots `productName`, `sizeName`, `colorName` — product data can change after purchase
- **`CartItem`** references `Inventory` directly, tying the cart to an exact SKU (product + size + color combo), which is the right approach for inventory reservation
- **`VerificationToken`** is needed for your `isVerified` field on `User` that's currently always `false`
