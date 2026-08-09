# Categories
```postgresql
INSERT INTO categories (
  "id",
  "name",
  "slug",
  "description",
  "sizeGuide",
  "isActive",
  "sortOrder",
  "deletedAt",
  "createdAt",
  "updatedAt"
)
VALUES
  (gen_random_uuid(), 'T-Shirts', 't-shirts', 'Short-sleeved casual tops in crewneck or V-neck styles, made from soft cotton or blends.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Tank Tops', 'tank-tops', 'Sleeveless tops ideal for layering or warm weather, available in various necklines.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Polo Shirts', 'polo-shirts', 'Collared casual shirts with a buttoned placket, offering a smart-casual look.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Cardigans', 'cardigans', 'Open-front knitted sweaters that button or zip, perfect for layering over tops.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Vests', 'vests', 'Sleeveless garment worn over shirts or alone, ranging from casual puffer styles to formal waistcoats.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Leggings', 'leggings', 'Stretchy, form-fitting pants made of spandex or cotton, ideal for activewear or casual comfort.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Joggers', 'joggers', 'Comfortable athletic pants with elastic cuffs and adjustable waist, suitable for lounging or sports.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Overalls', 'overalls', 'One-piece garment combining trousers with a front bib and shoulder straps, often made of denim.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Jumpsuits', 'jumpsuits', 'Full-length one-piece outfit combining top and trousers, popular for parties and casual wear.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Coats', 'coats', 'Heavy outerwear designed for cold weather, including trench, peacoat, and puffer styles.', NULL, true, 0, NULL, NOW(), NOW()),

  (gen_random_uuid(), 'Shirts', 'shirts', 'Casual and formal tops with buttons or pullover styles, available in various fabrics and fits.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Shorts', 'shorts', 'Comfortable bottom wear with short length, ideal for warm weather and casual outings.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Skirts', 'skirts', 'Feminine lower garments that drape from the waist, available in mini, midi, or maxi lengths.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Trousers', 'trousers', 'Full-length tailored pants suitable for office, casual, or formal occasions.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Dresses', 'dresses', 'One-piece garments combining bodice and skirt, perfect for parties, work, or everyday wear.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Jackets', 'jackets', 'Outerwear that adds style and warmth, ranging from lightweight to insulated designs.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Sweaters', 'sweaters', 'Knitted upper body garments designed for warmth and coziness during colder months.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Hoodies', 'hoodies', 'Casual sweatshirts with a hood and often a front pocket, great for relaxed wear.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Jeans', 'jeans', 'Durable denim pants in various cuts and washes, a timeless wardrobe staple.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Blouses', 'blouses', 'Lightweight, often flowy tops designed for women, suitable for both casual and formal settings.', NULL, true, 0, NULL, NOW(), NOW()),

  (gen_random_uuid(), 'Crop Tops', 'crop-tops', 'Short tops that end above the waist, perfect for pairing with high-waisted bottoms.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Bodysuits', 'bodysuits', 'One-piece form-fitting tops that snap at the crotch, offering a smooth tucked-in look.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Halter Tops', 'halter-tops', 'Sleeveless tops with straps that tie or fasten behind the neck, leaving shoulders bare.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Swimsuits', 'swimsuits', 'Garments designed for swimming and beach activities, available in one-piece or bikini styles.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Sweatpants', 'sweatpants', 'Comfortable elastic-waist pants made of soft fleece or cotton, ideal for lounging or workouts.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Cargo Pants', 'cargo-pants', 'Casual pants with multiple large pockets on the sides, offering utility and a relaxed fit.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Chinos', 'chinos', 'Lightweight cotton trousers with a tailored fit, suitable for smart-casual occasions.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Pajamas', 'pajamas', 'Loose-fitting sleepwear sets consisting of a top and bottom, made from breathable fabrics.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Robes', 'robes', 'Flowing open-front garments worn over sleepwear or after bathing, often made of cotton or silk.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Kimonos', 'kimonos', 'Lightweight, wide-sleeved outerwear inspired by traditional Japanese garments, great for layering.', NULL, true, 0, NULL, NOW(), NOW()),

  (gen_random_uuid(), 'Tunics', 'tunics', 'Longer tops that extend to hip or thigh length, often worn over leggings or skinny pants.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Blazers', 'blazers', 'Structured, tailored jackets often worn for professional or smart-casual occasions.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Rompers', 'rompers', 'One-piece short outfit combining a top with shorts, popular for summer and casual events.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Playsuits', 'playsuits', 'Similar to rompers, often looser and more playful, with varying sleeve and neckline styles.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Skorts', 'skorts', 'Hybrid garment that looks like a skirt from the front but has built-in shorts underneath.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Culottes', 'culottes', 'Wide-legged cropped pants that resemble a skirt, offering a flowy and comfortable fit.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Capris', 'capris', 'Pants that end between the knee and ankle, ideal for transitional weather or casual wear.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Yoga Pants', 'yoga-pants', 'Stretchy, high-waisted pants designed for yoga, exercise, or everyday comfort.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Biker Shorts', 'biker-shorts', 'Tight-fitting shorts that extend to mid-thigh, originally for cycling but now worn as casual wear.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Henley Shirts', 'henley-shirts', 'Casual collarless shirts with a buttoned placket at the neckline, often long-sleeved.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Peplum Tops', 'peplum-tops', 'Fitted tops with a flared ruffle or waistband at the bottom, creating an hourglass silhouette.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Windbreakers', 'windbreakers', 'Lightweight outerwear designed to resist wind and light rain, often packable.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Bomber Jackets', 'bomber-jackets', 'Short, zip-front jackets with elastic cuffs and hem, originally worn by pilots.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Turtlenecks', 'turtlenecks', 'Sweaters or tops with a high, folded collar that covers the neck, providing warmth.', NULL, true, 0, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Ponchos', 'ponchos', 'Simple, sleeveless outer garments with an opening for the head, offering a relaxed drape.', NULL, true, 0, NULL, NOW(), NOW());
```

# Designs
```postgresql
INSERT INTO designs (
  "id",
  "name",
  "slug",
  "description",
  "isActive",
  "deletedAt",
  "createdAt",
  "updatedAt"
)
VALUES
  (gen_random_uuid(), 'Striped Pattern', 'striped-pattern', 'Vertical or horizontal stripes that add a classic, sporty, or nautical touch to garments.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Checked Pattern', 'checked-pattern', 'Grid-like pattern formed by crossed horizontal and vertical lines, including plaid and gingham.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Crew Neck', 'crew-neck', 'Round, close-fitting neckline without a collar, commonly found on t-shirts and sweaters.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'V-Neck', 'v-neck', 'Neckline shaped like a V, creating a lengthening effect on the neck and chest area.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Pilgrim Collar', 'pilgrim-collar', 'Wide, flat collar with rounded corners, often seen on retro or vintage-style blouses and dresses.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Cloverleaf Collar', 'cloverleaf-collar', 'Scalloped, leaf-shaped collar that adds a delicate and feminine detail to tops.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Dry Fit', 'dry-fit', 'Moisture-wicking fabric design that pulls sweat away from the body, ideal for activewear.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Baggy Fit', 'baggy-fit', 'Loose, oversized cut providing maximum comfort and a relaxed streetwear aesthetic.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Slim Fit', 'slim-fit', 'Tailored, close-to-the-body cut that follows natural contours for a modern, streamlined look.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Floral Pattern', 'floral-pattern', 'Decorative design featuring flower motifs, ranging from subtle prints to bold botanical statements.', true, NULL, NOW(), NOW()),

  (gen_random_uuid(), 'Henley Neck', 'henley-neck', 'Round neckline with a short buttoned placket (2-5 buttons), often seen on casual tops.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Turtleneck', 'turtleneck', 'High, folded collar that covers the neck completely, providing warmth and a sleek silhouette.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Mock Neck', 'mock-neck', 'Shorter version of a turtleneck, standing upright but not folding over.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Peter Pan Collar', 'peter-pan-collar', 'Rounded, flat collar that lies against the garment, creating a youthful and vintage look.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Shawl Collar', 'shawl-collar', 'Continuous rolled collar that extends from the back of the neck down the front, typical on cardigans and robes.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Houndstooth', 'houndstooth', 'Classic two-tone textile pattern with broken checks or jagged teeth, often in black and white.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Polka Dot', 'polka-dot', 'Pattern consisting of an array of filled circles, giving a playful and retro feel.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Tie-Dye', 'tie-dye', 'Swirling, psychedelic pattern created by folding and tying fabric before dyeing.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Camouflage', 'camouflage', 'Mottled pattern using earthy or military colors to blend into surroundings.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Oversized Fit', 'oversized-fit', 'Extremely loose and boxy cut, often extending beyond standard sizing for a fashion-forward look.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Relaxed Fit', 'relaxed-fit', 'Comfortable cut with extra room through the chest and waist, less fitted than classic.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Drop Shoulder', 'drop-shoulder', 'Design where the sleeve seam sits lower than the natural shoulder, creating a casual, slouchy look.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Raglan Sleeve', 'raglan-sleeve', 'Sleeve that extends in one piece to the collar, with diagonal seams from underarm to neckline.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Batik Print', 'batik-print', 'Wax-resist dyeing technique producing intricate, organic patterns, often featuring dots and lines.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Paisley', 'paisley', 'Teardrop-shaped botanical motif with intricate curved details, inspired by traditional Persian designs.', true, NULL, NOW(), NOW()),

  (gen_random_uuid(), 'Mandarin Collar', 'mandarin-collar', 'Short, stand-up collar that does not fold, originating from traditional Asian clothing.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Lapel Collar', 'lapel-collar', 'Folded flaps on the front of a jacket or blouse, common on blazers and formal coats.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Keyhole Neck', 'keyhole-neck', 'Neckline with a small teardrop or circular cutout below the main opening, often fastened with a button.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Sweetheart Neckline', 'sweetheart-neckline', 'Neckline shaped like the top of a heart, creating a romantic and feminine look.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Chevron Pattern', 'chevron-pattern', 'Inverted V-shaped stripes that create a zigzag effect, often used in modern or geometric designs.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Herringbone', 'herringbone', 'V-shaped weaving pattern resembling fish bones, commonly found in suits and outerwear.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Argyle', 'argyle', 'Diamond-shaped pattern with overlapping lines, typically seen on sweaters and socks.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Abstract Print', 'abstract-print', 'Non-representational artistic patterns using shapes, colors, and lines for a unique look.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Colorblock', 'colorblock', 'Design featuring solid contrasting panels of color, creating bold visual sections.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Thermal Fabric', 'thermal-fabric', 'Honeycomb or waffle knit texture that traps heat, commonly used in long-sleeved casual tops.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Ribbed Knit', 'ribbed-knit', 'Vertical textured ridges created by alternating knit and purl stitches, offering stretch and structure.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Cropped Length', 'cropped-length', 'Garment cut shorter than standard, ending above the waist or hip.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'High-Low Hem', 'high-low-hem', 'Hemline that is shorter in the front and longer in the back, adding drama and movement.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Asymmetrical', 'asymmetrical', 'Design with uneven or irregular hemline, neckline, or overall shape for an edgy look.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Cutout Design', 'cutout-design', 'Strategic openings or gaps in the fabric, often at the shoulders, waist, or back.', true, NULL, NOW(), NOW()),

  (gen_random_uuid(), 'Off-Shoulder', 'off-shoulder', 'Neckline that exposes the shoulders while keeping sleeves or bodice covering the upper arms.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Boat Neck', 'boat-neck', 'Wide, horizontal neckline that runs from shoulder to shoulder, creating a graceful, elegant look.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Seersucker', 'seersucker', 'Lightweight fabric with a puckered, striped texture that resists clinging to the skin, ideal for summer.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Embroidered Design', 'embroidered-design', 'Decorative stitching applied to fabric, creating raised or textured patterns, motifs, or initials.', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Ruffle Detail', 'ruffle-detail', 'Strip of fabric gathered along one edge and sewn to the garment, adding volume and a playful touch.', true, NULL, NOW(), NOW());
```

# Colors
```postgresql
INSERT INTO colors ("id", "name", "hexCode", "isActive", "deletedAt", "createdAt", "updatedAt")
VALUES
  (gen_random_uuid(), 'Red', 'FF0000', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Green', '008000', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Blue', '0000FF', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Yellow', 'FFFF00', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Black', '000000', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'White', 'FFFFFF', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Purple', '800080', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Orange', 'FFA500', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Pink', 'FFC0CB', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Brown', 'A52A2A', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Gray', '808080', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Cyan', '00FFFF', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Magenta', 'FF00FF', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Lime', '00FF00', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Maroon', '800000', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Navy', '000080', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Olive', '808000', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Teal', '008080', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Silver', 'C0C0C0', true, NULL, NOW(), NOW()),
  (gen_random_uuid(), 'Gold', 'FFD700', true, NULL, NOW(), NOW());
```

# Products
```postgresql
INSERT INTO products ( "id", "name", "slug", "description", "categoryId", "sizeGuide", "gender", "ageGroup", "brand", "material", "careInstructions", "discountPercentage", "isFeatured", "status", "metaTitle", "metaDescription", "deletedAt", "createdAt", "updatedAt" ) VALUES ('a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', 'Classic Cotton T-Shirt', 'classic-cotton-t-shirt', 'A soft, breathable everyday t-shirt made from 100% combed cotton. Perfect for casual wear or layering.', '1d4b016c-f21a-4e56-bb08-57dbec757b93', NULL, 'UNISEX', 'ADULT', 'Wichithra', '100% Combed Cotton', 'Machine wash cold with like colors. Tumble dry low. Do not bleach.', 0, false, 'AVAILABLE', 'Classic Cotton T-Shirt | Buy Online | Wichithra', 'Soft and breathable 100% combed cotton t-shirt for everyday comfort.', NULL, '2026-05-29 12:00:00+05:30', '2026-05-29 12:00:00+05:30'), 

('b0eebc99-9c0b-4ef8-bb6d-6bb9bd380a22', 'Slim Fit Denim Jeans', 'slim-fit-denim-jeans', 'Classic five-pocket jeans with a modern slim fit. Durable denim with a hint of stretch for all-day comfort.', '9dbf615d-5ae3-4d7c-9f28-cf452e450c23', NULL, 'UNISEX', 'ADULT', 'Denim Co.', '98% Cotton, 2% Elastane', 'Machine wash cold inside out. Tumble dry medium. Wash with similar colors.', 0, false, 'AVAILABLE', 'Slim Fit Denim Jeans | Wichithra', 'Modern slim-fit jeans with stretch denim – comfortable and stylish for daily wear.', NULL, '2026-05-29 12:00:00+05:30', '2026-05-29 12:00:00+05:30'), 

('c0eebc99-9c0b-4ef8-bb6d-6bb9bd380a33', 'Floral Print Summer Dress', 'floral-print-summer-dress', 'Lightweight chiffon dress with a vibrant floral pattern. Features a flowy skirt and short sleeves – ideal for warm days.', 'dae90135-bc99-41df-a2d7-b8cb0c73bfc2', NULL, 'FEMALE', 'ADULT', 'Bloom & Co.', '100% Polyester Chiffon', 'Hand wash cold or use delicate cycle. Hang dry. Do not iron directly on print.', 0, false, 'AVAILABLE', 'Floral Print Summer Dress | Feminine & Flowy | Wichithra', 'Vibrant floral chiffon dress with a flowy skirt – perfect for summer outings.', NULL, '2026-05-29 12:00:00+05:30', '2026-05-29 12:00:00+05:30'),

 ('d0eebc99-9c0b-4ef8-bb6d-6bb9bd380a44', 'Heavyweight Hoodie', 'heavyweight-hoodie', 'Super soft fleece hoodie with a kangaroo pocket and adjustable drawstring hood. Great for lounging or active days.', '76d46ff9-7324-4923-aed2-9eba3e74fca6', NULL, 'UNISEX', 'ADULT', 'Urban Warmth', '80% Cotton, 20% Polyester', 'Machine wash warm. Tumble dry low. Do not use fabric softener.', 0, false, 'AVAILABLE', 'Heavyweight Hoodie | Cozy Fleece | Wichithra', 'Ultra-soft fleece hoodie with adjustable hood – your go‑to for comfort.', NULL, '2026-05-29 12:00:00+05:30', '2026-05-29 12:00:00+05:30'), ('e0eebc99-9c0b-4ef8-bb6d-6bb9bd380a55', 'Lightweight Bomber Jacket', 'lightweight-bomber-jacket', 'Classic bomber silhouette with a water‑resistant shell, ribbed cuffs, and a zip front. Perfect for transitional weather.', 'd0825318-5425-4d95-bacd-6bb91f240ae4', NULL, 'UNISEX', 'ADULT', 'Flight Style', '100% Nylon', 'Spot clean only. Machine wash cold on gentle cycle if needed. Line dry.', 0, false, 'AVAILABLE', 'Lightweight Bomber Jacket | Water‑Resistant | Wichithra', 'Classic bomber jacket with water‑resistant nylon shell and ribbed trims.', NULL, '2026-05-29 12:00:00+05:30', '2026-05-29 12:00:00+05:30'), ('f0eebc99-9c0b-4ef8-bb6d-6bb9bd380a66', 'Tailored Blazer', 'tailored-blazer', 'Sharp single‑breasted blazer with a notched lapel and subtle structure. Elevates any smart‑casual or office outfit.', '68a1d048-0ad5-4be1-a0b2-8d857f808bb4', NULL, 'UNISEX', 'ADULT', 'Executive Edge', '70% Polyester, 30% Viscose', 'Dry clean only. Low iron if needed.', 0, false, 'AVAILABLE', 'Tailored Blazer | Smart Casual | Wichithra', 'Sharp single‑breasted blazer for office or evening – structured yet comfortable.', NULL, '2026-05-29 12:00:00+05:30', '2026-05-29 12:00:00+05:30'), ('a1eebc99-9c0b-4ef8-bb6d-6bb9bd380a77', 'Stretch Chino Trousers', 'stretch-chino-trousers', 'Versatile chinos with a hint of stretch for freedom of movement. Classic fit, ideal for both office and weekend.', 'cda13fe7-800b-40a3-b8f4-1c479c584c1d', NULL, 'UNISEX', 'ADULT', 'Modern Fit', '97% Cotton, 3% Elastane', 'Machine wash cold. Tumble dry low. Warm iron if needed.', 0, false, 'AVAILABLE', 'Stretch Chino Trousers | Comfort Fit | Wichithra', 'Classic chinos with stretch – perfect for work or casual days.', NULL, '2026-05-29 12:00:00+05:30', '2026-05-29 12:00:00+05:30'), ('b1eebc99-9c0b-4ef8-bb6d-6bb9bd380a88', 'Cable Knit Sweater', 'cable-knit-sweater', 'Classic cable‑knit texture with a relaxed fit. Warm and breathable – a timeless winter essential.', '0017cee6-7e21-4bf1-987d-2f03ae4c16b4', NULL, 'UNISEX', 'ADULT', 'Heritage Knits', '100% Acrylic', 'Machine wash cold on gentle. Lay flat to dry. Do not iron.', 0, false, 'AVAILABLE', 'Cable Knit Sweater | Winter Essential | Wichithra', 'Timeless cable‑knit sweater – warm, breathable, and perfect for layering.', NULL, '2026-05-29 12:00:00+05:30', '2026-05-29 12:00:00+05:30'), ('c1eebc99-9c0b-4ef8-bb6d-6bb9bd380a99', 'High-Waist Leggings', 'high-waist-leggings', 'Buttery‑soft, squat‑proof leggings with a wide high‑waistband. Perfect for yoga, gym, or everyday wear.', 'c0f6bd52-6b71-49f4-8a6a-2e21ad81f81c', NULL, 'FEMALE', 'ADULT', 'Active Flow', '75% Nylon, 25% Spandex', 'Machine wash cold with like colors. Do not use fabric softener. Tumble dry low.', 0, false, 'AVAILABLE', 'High-Waist Leggings | Squat‑Proof | Wichithra', 'Buttery‑soft high‑waist leggings – ideal for yoga, gym, or all‑day comfort.', NULL, '2026-05-29 12:00:00+05:30', '2026-05-29 12:00:00+05:30'), ('d1eebc99-9c0b-4ef8-bb6d-6bb9bd380a00', 'Performance Polo Shirt', 'performance-polo-shirt', 'Moisture‑wicking polo with a classic three‑button placket and ribbed collar. Keeps you cool and sharp.', 'fe799ed7-7dc2-4110-b776-6ed3f56d4b9e', NULL, 'MALE', 'ADULT', 'Sportech', '100% Pique Cotton', 'Machine wash cold. Do not bleach. Tumble dry low. Cool iron if needed.', 0, false, 'AVAILABLE', 'Performance Polo Shirt | Moisture‑Wicking | Wichithra', 'Breathable pique cotton polo – moisture‑wicking and perfect for work or golf.', NULL, '2026-05-29 12:00:00+05:30', '2026-05-29 12:00:00+05:30');
```

```postgresql
INSERT INTO products (
    "id", "name", "slug", "description", "categoryId", "sizeGuide", "gender", "ageGroup", "brand", "material",
    "careInstructions", "discountPercentage", "isFeatured", "status", "metaTitle", "metaDescription",
    "deletedAt", "createdAt", "updatedAt"
) VALUES
    ('a2eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', 'Relaxed Fit Joggers', 'relaxed-fit-joggers', 'Everyday joggers with an elastic waistband, tapered legs, and soft fleece lining for maximum comfort.', '2912a20d-78a3-49ff-8b39-86918fa9fbd9', NULL, 'UNISEX', 'ADULT', 'Cozy Wear', '60% Cotton, 40% Polyester', 'Machine wash cold. Tumble dry low. Do not iron on print.', 0, false, 'AVAILABLE', 'Relaxed Fit Joggers | Everyday Comfort | Wichithra', 'Soft fleece joggers with tapered fit – perfect for lounging or running errands.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('b2eebc99-9c0b-4ef8-bb6d-6bb9bd380a22', 'Cotton Blend Blouse', 'cotton-blend-blouse', 'Lightweight blouse with a relaxed fit, featuring subtle pintucks and a round neckline. Ideal for office or casual days.', 'df535673-efef-4846-9820-d4e1e7f42350', NULL, 'FEMALE', 'ADULT', 'Elegant Basics', '70% Cotton, 30% Polyester', 'Machine wash gentle cycle. Hang dry. Warm iron if needed.', 0, false, 'AVAILABLE', 'Cotton Blend Blouse | Lightweight & Breathable | Wichithra', 'Versatile blouse with pintuck details – comfortable and chic for work or weekend.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('c2eebc99-9c0b-4ef8-bb6d-6bb9bd380a33', 'Cargo Utility Pants', 'cargo-utility-pants', 'Durable cargo pants with multiple pockets, a relaxed fit, and a adjustable waistband. Built for adventure.', '293e4a3a-e170-4e68-92ef-dd3df4f9c954', NULL, 'MALE', 'ADULT', 'Trail Gear', '100% Cotton Twill', 'Machine wash cold. Tumble dry medium. Do not bleach.', 0, false, 'AVAILABLE', 'Cargo Utility Pants | Durable & Functional | Wichithra', 'Classic cargo pants with utility pockets – rugged and comfortable for outdoor wear.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('d2eebc99-9c0b-4ef8-bb6d-6bb9bd380a44', 'Halter Neck Top', 'halter-neck-top', 'Sleeveless top with a tie‑back halter neckline, made from lightweight woven fabric. Perfect for summer parties.', '25c1dc3d-a650-4e3a-a429-0a86632bcbf7', NULL, 'FEMALE', 'ADULT', 'Summer Vibes', '100% Rayon', 'Hand wash cold. Line dry. Do not twist or wring.', 0, false, 'AVAILABLE', 'Halter Neck Top | Chic Summer Style | Wichithra', 'Elegant halter top with tie back – breezy and stylish for warm days.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('e2eebc99-9c0b-4ef8-bb6d-6bb9bd380a55', 'Cropped Jean Jacket', 'cropped-jean-jacket', 'Classic denim jacket with a cropped length, button front, and chest pockets. Adds a retro touch to any outfit.', '523126a8-ef40-456a-9119-005058ffa989', NULL, 'FEMALE', 'ADULT', 'Denim Revival', '99% Cotton, 1% Elastane', 'Machine wash cold inside out. Tumble dry low. Wash with like colors.', 0, false, 'AVAILABLE', 'Cropped Jean Jacket | Vintage Denim Style | Wichithra', 'Cropped denim jacket with classic details – a timeless layering piece.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('f2eebc99-9c0b-4ef8-bb6d-6bb9bd380a66', 'Pleated Midi Skirt', 'pleated-midi-skirt', 'Elegant pleated skirt that falls below the knee. Made from flowy fabric with an elastic waistband for comfort.', 'dbd07820-208b-41a5-a43c-18b56bbdde67', NULL, 'FEMALE', 'ADULT', 'Grace & Flow', '100% Polyester Georgette', 'Hand wash cold. Do not tumble dry. Low iron on reverse side.', 0, false, 'AVAILABLE', 'Pleated Midi Skirt | Flowy & Feminine | Wichithra', 'Classic pleated midi skirt – perfect for work, dates, or brunch.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('a3eebc99-9c0b-4ef8-bb6d-6bb9bd380a77', 'Henley Long Sleeve', 'henley-long-sleeve', 'Casual henley shirt with a 3‑button placket and soft, brushed fabric. Great for layering or wearing alone.', 'a5fe2685-dea3-42af-92a9-181bd3c997fb', NULL, 'UNISEX', 'ADULT', 'Heritage Basics', '100% Cotton', 'Machine wash warm. Tumble dry low. Do not use bleach.', 0, false, 'AVAILABLE', 'Henley Long Sleeve | Classic Casual | Wichithra', 'Soft cotton henley with button placket – effortless everyday style.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('b3eebc99-9c0b-4ef8-bb6d-6bb9bd380a88', 'Kimono Cardigan', 'kimono-cardigan', 'Lightweight open-front cardigan with wide sleeves and a floral print. Ideal for layering over tanks or dresses.', 'bff092d5-39dd-4631-9b49-ff1f46e3a732', NULL, 'FEMALE', 'ADULT', 'Boho Chic', '100% Viscose', 'Machine wash delicate. Hang dry. Cool iron if needed.', 0, false, 'AVAILABLE', 'Kimono Cardigan | Flowy Layer | Wichithra', 'Artistic kimono cardigan – adds a bohemian touch to any outfit.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('c3eebc99-9c0b-4ef8-bb6d-6bb9bd380a99', 'High-Waist Shorts', 'high-waist-shorts', 'Stretchy high‑waist shorts with a raw hem and rolled cuffs. Perfect for casual summer days or workouts.', 'ce36c704-76db-4684-98ea-0f3e19a8ef57', NULL, 'FEMALE', 'ADULT', 'Active Life', '95% Cotton, 5% Spandex', 'Machine wash cold. Tumble dry low. Do not iron directly on print.', 0, false, 'AVAILABLE', 'High-Waist Shorts | Summer Essential | Wichithra', 'Comfortable high‑waist shorts with stretch – great for warm weather.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('d3eebc99-9c0b-4ef8-bb6d-6bb9bd380a00', 'Seamless Bodysuit', 'seamless-bodysuit', 'Buttery‑soft seamless bodysuit with a snap closure at the crotch. Fits like a second skin – perfect for layering.', 'bf9f5cbf-5289-49d4-823e-6c1489c7afe0', NULL, 'FEMALE', 'ADULT', 'Essentials', '72% Nylon, 28% Spandex', 'Machine wash cold gentle. Lay flat to dry. Do not use fabric softener.', 0, false, 'AVAILABLE', 'Seamless Bodysuit | Smooth Layering | Wichithra', 'Stretchy seamless bodysuit – invisible under clothes and ultra comfortable.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('e3eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', 'Puffer Vest', 'puffer-vest', 'Lightweight quilted vest with a stand collar and zip pockets. Provides core warmth without restricting arms.', '69ccbe87-7241-4cfe-a64a-1d6b4feb2ad6', NULL, 'UNISEX', 'ADULT', 'Arctic Edge', '100% Polyester Shell, 100% Nylon Lining', 'Spot clean. Machine wash cold on gentle if needed. Hang dry.', 0, false, 'AVAILABLE', 'Puffer Vest | Lightweight Warmth | Wichithra', 'Quilted vest for layering – keeps you warm without bulk.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('f3eebc99-9c0b-4ef8-bb6d-6bb9bd380a22', 'Midi Wrap Dress', 'midi-wrap-dress', 'Flattering wrap dress with a V‑neckline, adjustable waist tie, and midi length. Effortlessly elegant.', 'dae90135-bc99-41df-a2d7-b8cb0c73bfc2', NULL, 'FEMALE', 'ADULT', 'Sofia Collection', '95% Polyester, 5% Spandex', 'Machine wash cold delicate. Hang dry. Low iron if needed.', 0, false, 'AVAILABLE', 'Midi Wrap Dress | Timeless Silhouette | Wichithra', 'Classic wrap dress – adjustable fit and perfect for any occasion.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('a4eebc99-9c0b-4ef8-bb6d-6bb9bd380a33', 'Linen Button-Up Shirt', 'linen-button-up-shirt', 'Breathable linen shirt with a relaxed fit, chest pocket, and button cuffs. Ideal for hot climates.', '81e46184-ffed-4ec4-b6ba-c247eca4ca19', NULL, 'UNISEX', 'ADULT', 'Natural Threads', '100% Linen', 'Machine wash cold gentle. Tumble dry low or line dry. Iron while damp.', 0, false, 'AVAILABLE', 'Linen Button-Up Shirt | Cool & Breathable | Wichithra', 'Pure linen shirt – lightweight and perfect for summer.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('b4eebc99-9c0b-4ef8-bb6d-6bb9bd380a44', 'Ribbed Turtleneck', 'ribbed-turtleneck', 'Fitted turtleneck with a ribbed texture and fold‑over collar. Made from soft cotton blend for warmth and style.', 'e62e4c0b-b4e0-43f4-b734-74c6cebbea9a', NULL, 'FEMALE', 'ADULT', 'Cozy Layers', '95% Cotton, 5% Elastane', 'Machine wash cold. Tumble dry low. Do not iron on ribbing.', 0, false, 'AVAILABLE', 'Ribbed Turtleneck | Sleek & Warm | Wichithra', 'Soft ribbed turtleneck – a winter wardrobe essential.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30'),
    ('c4eebc99-9c0b-4ef8-bb6d-6bb9bd380a55', 'Biker Shorts 2-Pack', 'biker-shorts-2-pack', 'Set of two high‑waist biker shorts in neutral colors. Moisture‑wicking and squat‑proof for workouts or lounging.', 'c0e5e053-5ff4-4c0d-ab6d-f0a25c8bb114', NULL, 'FEMALE', 'ADULT', 'Active Core', '78% Nylon, 22% Spandex', 'Machine wash cold. Do not use fabric softener. Tumble dry low.', 0, false, 'AVAILABLE', 'Biker Shorts 2-Pack | Squat‑Proof | Wichithra', 'Value pack of high‑performance biker shorts – perfect for gym or leisure.', NULL, '2026-05-29 14:00:00+05:30', '2026-05-29 14:00:00+05:30');
```

```postgresql
INSERT INTO products (
    "id", "name", "slug", "description", "categoryId", "sizeGuide", "gender", "ageGroup", "brand", "material",
    "careInstructions", "discountPercentage", "isFeatured", "status", "metaTitle", "metaDescription",
    "deletedAt", "createdAt", "updatedAt"
) VALUES
    ('d4eebc99-9c0b-4ef8-bb6d-6bb9bd380a66', 'Cotton Poplin Shirt', 'cotton-poplin-shirt', 'Crisp cotton poplin shirt with a classic button‑down collar and chest pocket. Easy‑care and wrinkle‑resistant.', '81e46184-ffed-4ec4-b6ba-c247eca4ca19', NULL, 'MALE', 'ADULT', 'Wichithra', '100% Cotton Poplin', 'Machine wash cold. Tumble dry low. Warm iron if needed.', 0, false, 'AVAILABLE', 'Cotton Poplin Shirt | Everyday Essential | Wichithra', 'Classic button‑down shirt – breathable, crisp, and versatile for work or weekend.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('e4eebc99-9c0b-4ef8-bb6d-6bb9bd380a77', 'Fleece Hoodie Dress', 'fleece-hoodie-dress', 'Oversized hoodie dress made from soft fleece, with a kangaroo pocket and hood. Casual and cozy.', 'dae90135-bc99-41df-a2d7-b8cb0c73bfc2', NULL, 'FEMALE', 'ADULT', 'Comfy Chic', '70% Cotton, 30% Polyester', 'Machine wash cold. Tumble dry low. Do not bleach.', 0, false, 'AVAILABLE', 'Fleece Hoodie Dress | Cozy & Stylish | Wichithra', 'Oversized fleece hoodie dress – perfect for lounging or streetwear.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('f4eebc99-9c0b-4ef8-bb6d-6bb9bd380a88', 'Seamless Training Tank', 'seamless-training-tank', 'Seamless tank top with racerback design and moisture‑wicking fabric. Ideal for high‑intensity workouts.', 'f504d21f-8019-43f5-8c40-c99304f13315', NULL, 'UNISEX', 'ADULT', 'Active Tech', '88% Nylon, 12% Spandex', 'Machine wash cold with like colors. Do not use fabric softener. Tumble dry low.', 0, false, 'AVAILABLE', 'Seamless Training Tank | Sweat‑Wicking | Wichithra', 'Seamless racerback tank – breathable and supportive for gym sessions.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('a5eebc99-9c0b-4ef8-bb6d-6bb9bd380a99', 'Wool Blend Peacoat', 'wool-blend-peacoat', 'Classic double‑breasted peacoat in a wool blend. Warm, structured, and timeless for cold weather.', 'aba8cd2f-fc99-4584-941a-9e59ffd7d5fb', NULL, 'UNISEX', 'ADULT', 'Heritage Outerwear', '70% Wool, 30% Polyester', 'Dry clean only. Low iron if needed.', 0, false, 'AVAILABLE', 'Wool Blend Peacoat | Timeless Winter Coat | Wichithra', 'Classic peacoat – warm, tailored, and perfect for chilly days.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('b5eebc99-9c0b-4ef8-bb6d-6bb9bd380a00', 'Lace Trim Cami', 'lace-trim-cami', 'Silky camisole with delicate lace trim at the neckline and hem. Great for layering or sleepwear.', 'df535673-efef-4846-9820-d4e1e7f42350', NULL, 'FEMALE', 'ADULT', 'Intimate Basics', '95% Polyester, 5% Elastane', 'Hand wash cold. Line dry. Do not bleach.', 0, false, 'AVAILABLE', 'Lace Trim Cami | Delicate Layering | Wichithra', 'Silky cami with lace details – feminine and versatile.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('c5eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', 'Cuffed Sweatpants', 'cuffed-sweatpants', 'Relaxed fit sweatpants with elastic cuffs at the ankles, side pockets, and a drawstring waist.', '6f78d815-df50-490b-9494-d1a5290517f3', NULL, 'UNISEX', 'ADULT', 'Lounge Life', '60% Cotton, 40% Polyester', 'Machine wash cold. Tumble dry low. Do not iron.', 0, false, 'AVAILABLE', 'Cuffed Sweatpants | Ultimate Comfort | Wichithra', 'Everyday sweatpants with cuffed ankles – cozy and stylish for home or errands.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('d5eebc99-9c0b-4ef8-bb6d-6bb9bd380a22', 'Ribbed Knit Cardigan', 'ribbed-knit-cardigan', 'Long cardigan with a chunky ribbed knit, open front, and patch pockets. Perfect for layering over any outfit.', '598af414-5cfc-45af-ba61-24fe22740ffe', NULL, 'FEMALE', 'ADULT', 'Cozy Knits', '100% Acrylic', 'Machine wash cold gentle. Lay flat to dry. Do not tumble dry.', 0, false, 'AVAILABLE', 'Ribbed Knit Cardigan | Chunky Layer | Wichithra', 'Long ribbed cardigan – soft, warm, and effortlessly casual.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('e5eebc99-9c0b-4ef8-bb6d-6bb9bd380a33', 'Performance Jogger Set', 'performance-jogger-set', 'Matching two‑piece set: moisture‑wicking zip‑up jacket and tapered joggers. Ideal for activewear or travel.', '2912a20d-78a3-49ff-8b39-86918fa9fbd9', NULL, 'UNISEX', 'ADULT', 'Sport Pro', '92% Polyester, 8% Spandex', 'Machine wash cold. Tumble dry low. Do not use fabric softener.', 0, false, 'AVAILABLE', 'Performance Jogger Set | Matching Activewear | Wichithra', 'Zip‑up jacket + joggers set – breathable and stylish for workouts or lounging.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('f5eebc99-9c0b-4ef8-bb6d-6bb9bd380a44', 'High-Neck Sleeveless Top', 'high-neck-sleeveless-top', 'Sleek high‑neck sleeveless top made from stretchy double knit fabric. Works well for layering or on its own.', '25c1dc3d-a650-4e3a-a429-0a86632bcbf7', NULL, 'FEMALE', 'ADULT', 'Modern Basics', '95% Cotton, 5% Elastane', 'Machine wash cold. Tumble dry low. Cool iron if needed.', 0, false, 'AVAILABLE', 'High-Neck Sleeveless Top | Sleek Layering | Wichithra', 'Stretchy high‑neck top – perfect under blazers or alone.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('a6eebc99-9c0b-4ef8-bb6d-6bb9bd380a55', 'Denim Overalls', 'denim-overalls', 'Classic denim overalls with a front bib, adjustable straps, and side pockets. Relaxed fit for a vintage look.', '0e1928d4-bd6d-43b4-ba54-be0b8041a6d6', NULL, 'FEMALE', 'ADULT', 'Denim Heritage', '100% Cotton Denim', 'Machine wash cold inside out. Tumble dry low. Wash with like colors.', 0, false, 'AVAILABLE', 'Denim Overalls | Vintage Style | Wichithra', 'Classic denim overalls – retro, durable, and comfortable.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('b6eebc99-9c0b-4ef8-bb6d-6bb9bd380a66', 'Crinkle Gauze Blouse', 'crinkle-gauze-blouse', 'Lightweight crinkle gauze blouse with a relaxed fit, roll‑up sleeves, and a V‑neck. Breezy and bohemian.', 'df535673-efef-4846-9820-d4e1e7f42350', NULL, 'FEMALE', 'ADULT', 'Boho Breeze', '100% Cotton Gauze', 'Machine wash cold gentle. Tumble dry low or line dry. Do not iron.', 0, false, 'AVAILABLE', 'Crinkle Gauze Blouse | Airy & Artistic | Wichithra', 'Boho crinkle gauze top – lightweight and perfect for summer.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('c6eebc99-9c0b-4ef8-bb6d-6bb9bd380a77', 'Lightweight Rain Jacket', 'lightweight-rain-jacket', 'Packable water‑resistant jacket with a hood and zippered pockets. Ideal for unexpected showers.', '0be8c233-0d88-442a-a203-90118e13f23c', NULL, 'UNISEX', 'ADULT', 'Weather Guard', '100% Nylon', 'Machine wash cold gentle. Hang dry. Do not iron.', 0, false, 'AVAILABLE', 'Lightweight Rain Jacket | Packable Protection | Wichithra', 'Compact rain jacket – water‑resistant and easy to carry.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('d6eebc99-9c0b-4ef8-bb6d-6bb9bd380a88', 'Satin Pajama Set', 'satin-pajama-set', 'Luxurious satin pajama set with a button‑up top and elastic‑waist shorts. Lightweight and breathable for sleeping.', '18991583-b86a-4861-ba46-b8a0f916e883', NULL, 'FEMALE', 'ADULT', 'Midnight Silk', '100% Polyester Satin', 'Hand wash cold. Line dry. Do not bleach or iron.', 0, false, 'AVAILABLE', 'Satin Pajama Set | Luxury Sleepwear | Wichithra', 'Silky satin pajamas – comfortable and elegant for a good night’s sleep.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('e6eebc99-9c0b-4ef8-bb6d-6bb9bd380a99', 'Utility Vest', 'utility-vest', 'Sleeveless utility vest with multiple pockets, a full zip front, and a relaxed fit. Great for outdoor adventures.', '69ccbe87-7241-4cfe-a64a-1d6b4feb2ad6', NULL, 'MALE', 'ADULT', 'Trail Master', '100% Cotton Canvas', 'Machine wash cold. Tumble dry medium. Do not iron on prints.', 0, false, 'AVAILABLE', 'Utility Vest | Outdoor Essential | Wichithra', 'Multi‑pocket utility vest – durable and functional for hiking or daily wear.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30'),
    ('f6eebc99-9c0b-4ef8-bb6d-6bb9bd380a00', 'Pleated Tennis Skirt', 'pleated-tennis-skirt', 'Sporty pleated skirt with built‑in shorts, moisture‑wicking fabric, and an elastic waistband. Perfect for court or casual.', '3b0ca3b5-9fde-4088-b2ad-7fb45d05eeb1', NULL, 'FEMALE', 'ADULT', 'Active Style', '90% Polyester, 10% Spandex', 'Machine wash cold. Tumble dry low. Do not use fabric softener.', 0, false, 'AVAILABLE', 'Pleated Tennis Skirt | Skort with Built‑In Shorts | Wichithra', 'Sporty skort – feminine pleats and active performance.', NULL, '2026-05-29 16:00:00+05:30', '2026-05-29 16:00:00+05:30');
```

# Variants
```postgresql
INSERT INTO product_variants (
    "id",
    "productId",
    "colorId",
    "sizeId",
    "sku",
    "sellingPrice",
    "costPrice",
    "isMainColor",
    "isActive",
    "createdAt",
    "updatedAt"
) VALUES
(
    gen_random_uuid(),
    'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',  -- Classic Cotton T-Shirt
    '4115386c-9ef6-4e68-928c-339392a4b708',  -- White
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'CLASSIC_COTTON_TSHIRT_WHITE_M',
    24.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b0eebc99-9c0b-4ef8-bb6d-6bb9bd380a22',  -- Slim Fit Denim Jeans
    '22cccb13-1730-4ead-ab24-002a60b57811',  -- Blue
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'SLIM_FIT_DENIM_JEANS_BLUE_L',
    49.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c0eebc99-9c0b-4ef8-bb6d-6bb9bd380a33',  -- Floral Print Summer Dress
    'eb0a594f-3a7e-4ca4-9d45-301826765b2a',  -- Pink
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'FLORAL_PRINT_SUMMER_DRESS_PINK_S',
    39.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'd0eebc99-9c0b-4ef8-bb6d-6bb9bd380a44',  -- Heavyweight Hoodie
    'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616',  -- Black
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'HEAVYWEIGHT_HOODIE_BLACK_XL',
    59.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c1eebc99-9c0b-4ef8-bb6d-6bb9bd380a99',  -- High-Waist Leggings
    '270151f9-044b-4a4b-bf88-36b13ff38c4f',  -- Gray
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'HIGH_WAIST_LEGGINGS_GRAY_M',
    34.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'd1eebc99-9c0b-4ef8-bb6d-6bb9bd380a00',  -- Performance Polo Shirt
    'ab54caa7-5502-45b6-8251-152adac60cff',  -- Navy
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'PERFORMANCE_POLO_SHIRT_NAVY_L',
    29.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a4eebc99-9c0b-4ef8-bb6d-6bb9bd380a33',  -- Linen Button-Up Shirt
    '4115386c-9ef6-4e68-928c-339392a4b708',  -- White
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'LINEN_BUTTON_UP_SHIRT_WHITE_M',
    44.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b1eebc99-9c0b-4ef8-bb6d-6bb9bd380a88',  -- Cable Knit Sweater
    '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d',  -- Green
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'CABLE_KNIT_SWEATER_GREEN_M',
    69.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a6eebc99-9c0b-4ef8-bb6d-6bb9bd380a55',  -- Denim Overalls
    '22cccb13-1730-4ead-ab24-002a60b57811',  -- Blue
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'DENIM_OVERALLS_BLUE_L',
    79.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c2eebc99-9c0b-4ef8-bb6d-6bb9bd380a33',  -- Cargo Utility Pants
    '92105889-7c4b-4779-8d34-695376e1b630',  -- Olive
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'CARGO_UTILITY_PANTS_OLIVE_XL',
    54.99, NULL, FALSE, TRUE, NOW(), NOW()
);
```

```postgresql
INSERT INTO product_variants (
    "id",
    "productId",
    "colorId",
    "sizeId",
    "sku",
    "sellingPrice",
    "costPrice",
    "isMainColor",
    "isActive",
    "createdAt",
    "updatedAt"
) VALUES
(
    gen_random_uuid(),
    'a1eebc99-9c0b-4ef8-bb6d-6bb9bd380a77',  -- Stretch Chino Trousers
    '0c30e6a5-5e10-4a96-88f2-6f9823bc77f5',  -- Silver
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'STRETCH_CHINO_TROUSERS_SILVER_L',
    59.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a2eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',  -- Relaxed Fit Joggers
    'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616',  -- Black
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'RELAXED_FIT_JOGGERS_BLACK_XL',
    44.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a3eebc99-9c0b-4ef8-bb6d-6bb9bd380a77',  -- Henley Long Sleeve
    '7f7efa06-efb1-455f-a2fe-06d66e43fb1c',  -- Red
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'HENLEY_LONG_SLEEVE_RED_M',
    34.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a5eebc99-9c0b-4ef8-bb6d-6bb9bd380a99',  -- Wool Blend Peacoat
    '3facc0f8-0db8-4d5f-b01e-4393b78b2feb',  -- Brown
    '550e8400-e29b-41d4-a716-446655440015',  -- XXL
    'WOOL_BLEND_PEACOAT_BROWN_XXL',
    129.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b2eebc99-9c0b-4ef8-bb6d-6bb9bd380a22',  -- Cotton Blend Blouse
    'eb0a594f-3a7e-4ca4-9d45-301826765b2a',  -- Pink
    '550e8400-e29b-41d4-a716-446655440010',  -- XS
    'COTTON_BLEND_BLOUSE_PINK_XS',
    29.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b3eebc99-9c0b-4ef8-bb6d-6bb9bd380a88',  -- Kimono Cardigan
    'dba5cd05-3622-4b87-b7c3-ca02011a2a1b',  -- Cyan
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'KIMONO_CARDIGAN_CYAN_S',
    49.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b4eebc99-9c0b-4ef8-bb6d-6bb9bd380a44',  -- Ribbed Turtleneck
    '709681ad-cde8-4af3-a986-642fec48e0e3',  -- Magenta
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'RIBBED_TURTLENECK_MAGENTA_M',
    39.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b5eebc99-9c0b-4ef8-bb6d-6bb9bd380a00',  -- Lace Trim Cami
    '4115386c-9ef6-4e68-928c-339392a4b708',  -- White
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'LACE_TRIM_CAMI_WHITE_S',
    19.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b6eebc99-9c0b-4ef8-bb6d-6bb9bd380a66',  -- Crinkle Gauze Blouse
    '5cc348b9-236e-454e-97a7-5bdfad371be1',  -- Orange
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'CRINKLE_GAUZE_BLOUSE_ORANGE_M',
    42.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c3eebc99-9c0b-4ef8-bb6d-6bb9bd380a99',  -- High-Waist Shorts
    '6662b302-0a77-42ff-a6ff-307f36f36151',  -- Teal
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'HIGH_WAIST_SHORTS_TEAL_M',
    27.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c4eebc99-9c0b-4ef8-bb6d-6bb9bd380a55',  -- Biker Shorts 2-Pack
    'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616',  -- Black
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'BIKER_SHORTS_2PACK_BLACK_S',
    34.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c5eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',  -- Cuffed Sweatpants
    '270151f9-044b-4a4b-bf88-36b13ff38c4f',  -- Gray
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'CUFFED_SWEATPANTS_GRAY_L',
    39.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c6eebc99-9c0b-4ef8-bb6d-6bb9bd380a77',  -- Lightweight Rain Jacket
    'd5bfe851-dbca-4207-9688-d215bd416cce',  -- Gold
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'LIGHTWEIGHT_RAIN_JACKET_GOLD_XL',
    69.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'd2eebc99-9c0b-4ef8-bb6d-6bb9bd380a44',  -- Halter Neck Top
    '5aa0990b-88c0-4883-b5d9-b631433d56fd',  -- Yellow
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'HALTER_NECK_TOP_YELLOW_S',
    24.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'd3eebc99-9c0b-4ef8-bb6d-6bb9bd380a00',  -- Seamless Bodysuit
    '482ae634-b4f7-45b8-9368-14f9aa425fec',  -- Purple
    '550e8400-e29b-41d4-a716-446655440010',  -- XS
    'SEAMLESS_BODYSUIT_PURPLE_XS',
    22.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'd4eebc99-9c0b-4ef8-bb6d-6bb9bd380a66',  -- Cotton Poplin Shirt
    '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d',  -- Green
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'COTTON_POPLIN_SHIRT_GREEN_L',
    49.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'd5eebc99-9c0b-4ef8-bb6d-6bb9bd380a22',  -- Ribbed Knit Cardigan
    'a1aa4626-3fec-4788-8b20-347a7eb6cb88',  -- Lime
    '550e8400-e29b-41d4-a716-446655440016',  -- XXXL
    'RIBBED_KNIT_CARDIGAN_LIME_XXXL',
    54.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'd6eebc99-9c0b-4ef8-bb6d-6bb9bd380a88',  -- Satin Pajama Set
    'e58b51ed-4a20-44a7-bfa8-c6b4bffda29a',  -- Maroon
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'SATIN_PAJAMA_SET_MAROON_M',
    45.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'e0eebc99-9c0b-4ef8-bb6d-6bb9bd380a55',  -- Lightweight Bomber Jacket
    '22cccb13-1730-4ead-ab24-002a60b57811',  -- Blue
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'LIGHTWEIGHT_BOMBER_JACKET_BLUE_XL',
    89.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'e2eebc99-9c0b-4ef8-bb6d-6bb9bd380a55',  -- Cropped Jean Jacket
    'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616',  -- Black
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'CROPPED_JEAN_JACKET_BLACK_S',
    69.99, NULL, FALSE, TRUE, NOW(), NOW()
);
```

```postgresql
INSERT INTO product_variants (
    "id",
    "productId",
    "colorId",
    "sizeId",
    "sku",
    "sellingPrice",
    "costPrice",
    "isMainColor",
    "isActive",
    "createdAt",
    "updatedAt"
) VALUES
-- 10 new products (first variants)
(
    gen_random_uuid(),
    'e3eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',  -- Puffer Vest
    'ab54caa7-5502-45b6-8251-152adac60cff',  -- Navy
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'PUFFER_VEST_NAVY_M',
    69.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'e4eebc99-9c0b-4ef8-bb6d-6bb9bd380a77',  -- Fleece Hoodie Dress
    'eb0a594f-3a7e-4ca4-9d45-301826765b2a',  -- Pink
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'FLEECE_HOODIE_DRESS_PINK_S',
    59.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'e5eebc99-9c0b-4ef8-bb6d-6bb9bd380a33',  -- Performance Jogger Set
    'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616',  -- Black
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'PERFORMANCE_JOGGER_SET_BLACK_L',
    89.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'e6eebc99-9c0b-4ef8-bb6d-6bb9bd380a99',  -- Utility Vest
    '92105889-7c4b-4779-8d34-695376e1b630',  -- Olive
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'UTILITY_VEST_OLIVE_XL',
    79.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'f0eebc99-9c0b-4ef8-bb6d-6bb9bd380a66',  -- Tailored Blazer
    '270151f9-044b-4a4b-bf88-36b13ff38c4f',  -- Gray
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'TAILORED_BLAZER_GRAY_L',
    129.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'f2eebc99-9c0b-4ef8-bb6d-6bb9bd380a66',  -- Pleated Midi Skirt
    'e58b51ed-4a20-44a7-bfa8-c6b4bffda29a',  -- Maroon
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'PLEATED_MIDI_SKIRT_MAROON_M',
    49.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'f3eebc99-9c0b-4ef8-bb6d-6bb9bd380a22',  -- Midi Wrap Dress
    '482ae634-b4f7-45b8-9368-14f9aa425fec',  -- Purple
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'MIDI_WRAP_DRESS_PURPLE_S',
    64.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'f4eebc99-9c0b-4ef8-bb6d-6bb9bd380a88',  -- Seamless Training Tank
    'a1aa4626-3fec-4788-8b20-347a7eb6cb88',  -- Lime
    '550e8400-e29b-41d4-a716-446655440010',  -- XS
    'SEAMLESS_TRAINING_TANK_LIME_XS',
    34.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'f5eebc99-9c0b-4ef8-bb6d-6bb9bd380a44',  -- High-Neck Sleeveless Top
    '6662b302-0a77-42ff-a6ff-307f36f36151',  -- Teal
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'HIGH_NECK_SLEEVELESS_TOP_TEAL_M',
    39.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'f6eebc99-9c0b-4ef8-bb6d-6bb9bd380a00',  -- Pleated Tennis Skirt
    '4115386c-9ef6-4e68-928c-339392a4b708',  -- White
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'PLEATED_TENNIS_SKIRT_WHITE_S',
    44.99, NULL, FALSE, TRUE, NOW(), NOW()
),

-- 10 additional variants for already‑used products (new color/size combos)
(
    gen_random_uuid(),
    'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',  -- Classic Cotton T-Shirt
    'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616',  -- Black
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'CLASSIC_COTTON_TSHIRT_BLACK_L',
    24.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b0eebc99-9c0b-4ef8-bb6d-6bb9bd380a22',  -- Slim Fit Denim Jeans
    '270151f9-044b-4a4b-bf88-36b13ff38c4f',  -- Gray
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'SLIM_FIT_DENIM_JEANS_GRAY_M',
    49.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c0eebc99-9c0b-4ef8-bb6d-6bb9bd380a33',  -- Floral Print Summer Dress
    '5aa0990b-88c0-4883-b5d9-b631433d56fd',  -- Yellow
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'FLORAL_PRINT_SUMMER_DRESS_YELLOW_M',
    39.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'd0eebc99-9c0b-4ef8-bb6d-6bb9bd380a44',  -- Heavyweight Hoodie
    'ab54caa7-5502-45b6-8251-152adac60cff',  -- Navy
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'HEAVYWEIGHT_HOODIE_NAVY_S',
    59.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c1eebc99-9c0b-4ef8-bb6d-6bb9bd380a99',  -- High-Waist Leggings
    'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616',  -- Black
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'HIGH_WAIST_LEGGINGS_BLACK_S',
    34.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'd1eebc99-9c0b-4ef8-bb6d-6bb9bd380a00',  -- Performance Polo Shirt
    '4115386c-9ef6-4e68-928c-339392a4b708',  -- White
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'PERFORMANCE_POLO_SHIRT_WHITE_M',
    29.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a4eebc99-9c0b-4ef8-bb6d-6bb9bd380a33',  -- Linen Button-Up Shirt
    '22cccb13-1730-4ead-ab24-002a60b57811',  -- Blue
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'LINEN_BUTTON_UP_SHIRT_BLUE_L',
    44.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b1eebc99-9c0b-4ef8-bb6d-6bb9bd380a88',  -- Cable Knit Sweater
    '3facc0f8-0db8-4d5f-b01e-4393b78b2feb',  -- Brown
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'CABLE_KNIT_SWEATER_BROWN_XL',
    69.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a6eebc99-9c0b-4ef8-bb6d-6bb9bd380a55',  -- Denim Overalls
    'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616',  -- Black
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'DENIM_OVERALLS_BLACK_M',
    79.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c2eebc99-9c0b-4ef8-bb6d-6bb9bd380a33',  -- Cargo Utility Pants
    '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d',  -- Green
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'CARGO_UTILITY_PANTS_GREEN_L',
    54.99, NULL, FALSE, TRUE, NOW(), NOW()
);
```

```postgresql
INSERT INTO product_variants (
    "id",
    "productId",
    "colorId",
    "sizeId",
    "sku",
    "sellingPrice",
    "costPrice",
    "isMainColor",
    "isActive",
    "createdAt",
    "updatedAt"
) VALUES
-- Variants for products with fewer than 3 variants so far (adding fresh combos)
(
    gen_random_uuid(),
    'a1eebc99-9c0b-4ef8-bb6d-6bb9bd380a77',  -- Stretch Chino Trousers
    '0c30e6a5-5e10-4a96-88f2-6f9823bc77f5',  -- Silver (already exists? yes but with L; now with XS)
    '550e8400-e29b-41d4-a716-446655440010',  -- XS
    'STRETCH_CHINO_TROUSERS_SILVER_XS',
    59.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a1eebc99-9c0b-4ef8-bb6d-6bb9bd380a77',  -- Stretch Chino Trousers
    'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616',  -- Black (new color for this product)
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'STRETCH_CHINO_TROUSERS_BLACK_XL',
    59.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a2eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',  -- Relaxed Fit Joggers
    '5aa0990b-88c0-4883-b5d9-b631433d56fd',  -- Yellow (new)
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'RELAXED_FIT_JOGGERS_YELLOW_S',
    44.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a2eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',  -- Relaxed Fit Joggers
    '6662b302-0a77-42ff-a6ff-307f36f36151',  -- Teal (new)
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'RELAXED_FIT_JOGGERS_TEAL_L',
    44.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a3eebc99-9c0b-4ef8-bb6d-6bb9bd380a77',  -- Henley Long Sleeve
    'd5bfe851-dbca-4207-9688-d215bd416cce',  -- Gold (new)
    '550e8400-e29b-41d4-a716-446655440010',  -- XS
    'HENLEY_LONG_SLEEVE_GOLD_XS',
    34.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a3eebc99-9c0b-4ef8-bb6d-6bb9bd380a77',  -- Henley Long Sleeve
    'a1aa4626-3fec-4788-8b20-347a7eb6cb88',  -- Lime (new)
    '550e8400-e29b-41d4-a716-446655440016',  -- XXXL
    'HENLEY_LONG_SLEEVE_LIME_XXXL',
    34.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a5eebc99-9c0b-4ef8-bb6d-6bb9bd380a99',  -- Wool Blend Peacoat
    '5cc348b9-236e-454e-97a7-5bdfad371be1',  -- Orange
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'WOOL_BLEND_PEACOAT_ORANGE_M',
    129.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'a5eebc99-9c0b-4ef8-bb6d-6bb9bd380a99',  -- Wool Blend Peacoat
    'eb0a594f-3a7e-4ca4-9d45-301826765b2a',  -- Pink (unusual but possible)
    '550e8400-e29b-41d4-a716-446655440015',  -- XXL
    'WOOL_BLEND_PEACOAT_PINK_XXL',
    129.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b2eebc99-9c0b-4ef8-bb6d-6bb9bd380a22',  -- Cotton Blend Blouse
    '709681ad-cde8-4af3-a986-642fec48e0e3',  -- Magenta
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'COTTON_BLEND_BLOUSE_MAGENTA_XL',
    29.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b2eebc99-9c0b-4ef8-bb6d-6bb9bd380a22',  -- Cotton Blend Blouse
    '7f7efa06-efb1-455f-a2fe-06d66e43fb1c',  -- Red
    '550e8400-e29b-41d4-a716-446655440010',  -- XS
    'COTTON_BLEND_BLOUSE_RED_XS',
    29.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b3eebc99-9c0b-4ef8-bb6d-6bb9bd380a88',  -- Kimono Cardigan
    '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d',  -- Green
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'KIMONO_CARDIGAN_GREEN_L',
    49.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b3eebc99-9c0b-4ef8-bb6d-6bb9bd380a88',  -- Kimono Cardigan
    '92105889-7c4b-4779-8d34-695376e1b630',  -- Olive
    '550e8400-e29b-41d4-a716-446655440016',  -- XXXL
    'KIMONO_CARDIGAN_OLIVE_XXXL',
    49.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b4eebc99-9c0b-4ef8-bb6d-6bb9bd380a44',  -- Ribbed Turtleneck
    'dba5cd05-3622-4b87-b7c3-ca02011a2a1b',  -- Cyan
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'RIBBED_TURTLENECK_CYAN_XL',
    39.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b4eebc99-9c0b-4ef8-bb6d-6bb9bd380a44',  -- Ribbed Turtleneck
    'e58b51ed-4a20-44a7-bfa8-c6b4bffda29a',  -- Maroon
    '550e8400-e29b-41d4-a716-446655440011',  -- S
    'RIBBED_TURTLENECK_MAROON_S',
    39.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b5eebc99-9c0b-4ef8-bb6d-6bb9bd380a00',  -- Lace Trim Cami
    '482ae634-b4f7-45b8-9368-14f9aa425fec',  -- Purple
    '550e8400-e29b-41d4-a716-446655440012',  -- M
    'LACE_TRIM_CAMI_PURPLE_M',
    19.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'b6eebc99-9c0b-4ef8-bb6d-6bb9bd380a66',  -- Crinkle Gauze Blouse
    '3facc0f8-0db8-4d5f-b01e-4393b78b2feb',  -- Brown
    '550e8400-e29b-41d4-a716-446655440013',  -- L
    'CRINKLE_GAUZE_BLOUSE_BROWN_L',
    42.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c3eebc99-9c0b-4ef8-bb6d-6bb9bd380a99',  -- High-Waist Shorts
    '0c30e6a5-5e10-4a96-88f2-6f9823bc77f5',  -- Silver
    '550e8400-e29b-41d4-a716-446655440016',  -- XXXL
    'HIGH_WAIST_SHORTS_SILVER_XXXL',
    27.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c4eebc99-9c0b-4ef8-bb6d-6bb9bd380a55',  -- Biker Shorts 2-Pack
    'd5bfe851-dbca-4207-9688-d215bd416cce',  -- Gold
    '550e8400-e29b-41d4-a716-446655440014',  -- XL
    'BIKER_SHORTS_2PACK_GOLD_XL',
    34.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c5eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',  -- Cuffed Sweatpants
    'eb0a594f-3a7e-4ca4-9d45-301826765b2a',  -- Pink
    '550e8400-e29b-41d4-a716-446655440010',  -- XS
    'CUFFED_SWEATPANTS_PINK_XS',
    39.99, NULL, FALSE, TRUE, NOW(), NOW()
),
(
    gen_random_uuid(),
    'c6eebc99-9c0b-4ef8-bb6d-6bb9bd380a77',  -- Lightweight Rain Jacket
    'a1aa4626-3fec-4788-8b20-347a7eb6cb88',  -- Lime
    '550e8400-e29b-41d4-a716-446655440010',  -- XS
    'LIGHTWEIGHT_RAIN_JACKET_LIME_XS',
    69.99, NULL, FALSE, TRUE, NOW(), NOW()
);
```

## Order, OrderItem, OrderAddress, OrderStatusHistory
```postgresql
-- =====================================================
-- Single order with 4 items, address, status history, payment
-- Explicitly generates UUID for ALL id columns
-- =====================================================

WITH
-- 1. Pick an existing user (any active user)
selected_user AS (
    SELECT "id" AS user_id
    FROM users
    WHERE "deletedAt" IS NULL
    LIMIT 1
),

-- 2. Define the 4 inventory items (data directly from CSVs)
order_items_data AS (
    SELECT
        i."id" AS inventory_id,
        pv."sku",
        pr."name" AS product_name,
        sz."name" AS size_name,
        cl."name" AS color_name,
        NULL::varchar AS image_url,
        1 AS quantity,
        pv."sellingPrice" AS unit_price,
        0 AS discount_amount
    FROM (VALUES
        ('03c9123e-e112-4583-95b6-8b1454590dbb'::uuid, '79119d27-bef1-4052-afef-6f220df90bf9'::uuid, 'b5eebc99-9c0b-4ef8-bb6d-6bb9bd380a00'::uuid, '4115386c-9ef6-4e68-928c-339392a4b708'::uuid, '550e8400-e29b-41d4-a716-446655440011'::uuid, 'LACE_TRIM_CAMI_WHITE_S', 19.99),
        ('046b324d-f7e3-40ce-8972-0b0b7db164d7'::uuid, '2ef665d0-78c5-46eb-8c3a-54fa4443d54f'::uuid, 'c2eebc99-9c0b-4ef8-bb6d-6bb9bd380a33'::uuid, '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d'::uuid, '550e8400-e29b-41d4-a716-446655440013'::uuid, 'CARGO_UTILITY_PANTS_GREEN_L', 54.99),
        ('078db6e4-b7b3-417b-9f39-ef4dc9b42c74'::uuid, '2af5a689-db40-4057-b83d-21575ff94818'::uuid, 'f5eebc99-9c0b-4ef8-bb6d-6bb9bd380a44'::uuid, '6662b302-0a77-42ff-a6ff-307f36f36151'::uuid, '550e8400-e29b-41d4-a716-446655440012'::uuid, 'HIGH_NECK_SLEEVELESS_TOP_TEAL_M', 39.99),
        ('0c7b2cf0-d2bd-4404-ba98-ed75e3be08fc'::uuid, '542e4b56-c5ea-46b5-8df3-aedaa09fdcdf'::uuid, 'b2eebc99-9c0b-4ef8-bb6d-6bb9bd380a22'::uuid, 'eb0a594f-3a7e-4ca4-9d45-301826765b2a'::uuid, '550e8400-e29b-41d4-a716-446655440010'::uuid, 'COTTON_BLEND_BLOUSE_PINK_XS', 29.99)
    ) AS t(inv_id, var_id, prod_id, col_id, sz_id, sku_code, price)
    JOIN inventory i ON i."id" = t.inv_id
    JOIN product_variants pv ON pv."id" = t.var_id
    JOIN products pr ON pr."id" = t.prod_id
    JOIN sizes sz ON sz."id" = t.sz_id
    JOIN colors cl ON cl."id" = t.col_id
),

-- 3. Compute order totals
order_totals AS (
    SELECT
        SUM(unit_price * quantity) AS subtotal,
        SUM(unit_price * quantity) * 0.05 AS tax_amount,
        0::decimal(10,2) AS shipping_fee,
        SUM(unit_price * quantity) + (SUM(unit_price * quantity) * 0.05) AS total_amount
    FROM order_items_data
),

-- 4. Insert the order with explicit UUID for "id"
inserted_order AS (
    INSERT INTO orders (
        "id",
        "orderNumber",
        "userId",
        "status",
        "paymentStatus",
        "subtotal",
        "discountAmount",
        "shippingFee",
        "taxAmount",
        "totalAmount",
        "notes",
        "cancelReason",
        "createdAt",
        "updatedAt"
    )
    SELECT
        gen_random_uuid(),
        'ORD-2026-TEST001'::varchar,
        su.user_id,
        'PENDING'::order_status,
        'PENDING'::payment_status,
        ot.subtotal,
        0,
        ot.shipping_fee,
        ot.tax_amount,
        ot.total_amount,
        'Test order with 4 items',
        NULL,
        NOW(),
        NOW()
    FROM selected_user su
    CROSS JOIN order_totals ot
    RETURNING "id"
),

-- 5. Insert order items (4 rows) – explicit id
inserted_order_items AS (
    INSERT INTO order_items (
        "id",
        "orderId",
        "inventoryId",
        "sku",
        "productName",
        "sizeName",
        "colorName",
        "imageUrl",
        "quantity",
        "unitPrice",
        "discountAmount",
        "totalPrice",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        io."id",
        oid.inventory_id,
        oid.sku,
        oid.product_name,
        oid.size_name,
        oid.color_name,
        oid.image_url,
        oid.quantity,
        oid.unit_price,
        oid.discount_amount,
        oid.unit_price * oid.quantity,
        NOW()
    FROM inserted_order io
    CROSS JOIN order_items_data oid
    RETURNING "id"
),

-- 6. Insert order address – explicit id
inserted_order_address AS (
    INSERT INTO order_addresses (
        "id",
        "orderId",
        "type",
        "houseNo",
        "addressLine1",
        "addressLine2",
        "city",
        "province",
        "zipcode",
        "country",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        io."id",
        'DELIVERY'::address_type,
        '42',
        'Main Street',
        'Apt 3B',
        'Colombo',
        'Western',
        '00100',
        'Sri Lanka',
        NOW()
    FROM inserted_order io
    RETURNING "id"
),

-- 7. Insert order status history – explicit id
inserted_status_history AS (
    INSERT INTO order_status_history (
        "id",
        "orderId",
        "status",
        "notes",
        "createdBy",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        io."id",
        'PENDING'::order_status,
        'Order created via test script',
        NULL,
        NOW()
    FROM inserted_order io
    RETURNING "id"
),

-- 8. Insert payment – explicit id
inserted_payment AS (
    INSERT INTO payments (
        "id",
        "orderId",
        "amount",
        "currency",
        "method",
        "status",
        "gatewayProvider",
        "gatewayTransactionId",
        "gatewayResponse",
        "failureReason",
        "paidAt",
        "refundedAt",
        "createdAt",
        "updatedAt"
    )
    SELECT
        gen_random_uuid(),
        io."id",
        ot.total_amount,
        'LKR',
        'CARD'::payment_method,
        'PENDING'::payment_status,
        'PayHere',
        'txn_test_' || gen_random_uuid()::text,
        '{"test": true}'::jsonb,
        NULL,
        NULL,
        NULL,
        NOW(),
        NOW()
    FROM inserted_order io
    CROSS JOIN order_totals ot
    RETURNING "id"
)

-- Final output: show what was inserted
SELECT 'Order inserted with ID: ' || "id"::text AS result FROM inserted_order
UNION ALL
SELECT 'Order items inserted: 4' WHERE EXISTS (SELECT 1 FROM inserted_order_items)
UNION ALL
SELECT 'Order address inserted' WHERE EXISTS (SELECT 1 FROM inserted_order_address)
UNION ALL
SELECT 'Status history inserted' WHERE EXISTS (SELECT 1 FROM inserted_status_history)
UNION ALL
SELECT 'Payment inserted' WHERE EXISTS (SELECT 1 FROM inserted_payment);
```

```postgresql
-- =====================================================
-- Insert 10 orders, each with 3 order items,
-- address, status history, and payment
-- Fixed ambiguous column reference
-- =====================================================

WITH
-- 1. List of available inventory items (use the same IDs as before)
available_inventory AS (
    SELECT
        i."id" AS inventory_id,
        pv."sku",
        pr."name" AS product_name,
        sz."name" AS size_name,
        cl."name" AS color_name,
        NULL::varchar AS image_url,
        pv."sellingPrice" AS unit_price
    FROM (VALUES
        ('03c9123e-e112-4583-95b6-8b1454590dbb'::uuid, '79119d27-bef1-4052-afef-6f220df90bf9'::uuid, 'b5eebc99-9c0b-4ef8-bb6d-6bb9bd380a00'::uuid, '4115386c-9ef6-4e68-928c-339392a4b708'::uuid, '550e8400-e29b-41d4-a716-446655440011'::uuid),
        ('046b324d-f7e3-40ce-8972-0b0b7db164d7'::uuid, '2ef665d0-78c5-46eb-8c3a-54fa4443d54f'::uuid, 'c2eebc99-9c0b-4ef8-bb6d-6bb9bd380a33'::uuid, '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d'::uuid, '550e8400-e29b-41d4-a716-446655440013'::uuid),
        ('078db6e4-b7b3-417b-9f39-ef4dc9b42c74'::uuid, '2af5a689-db40-4057-b83d-21575ff94818'::uuid, 'f5eebc99-9c0b-4ef8-bb6d-6bb9bd380a44'::uuid, '6662b302-0a77-42ff-a6ff-307f36f36151'::uuid, '550e8400-e29b-41d4-a716-446655440012'::uuid),
        ('0c7b2cf0-d2bd-4404-ba98-ed75e3be08fc'::uuid, '542e4b56-c5ea-46b5-8df3-aedaa09fdcdf'::uuid, 'b2eebc99-9c0b-4ef8-bb6d-6bb9bd380a22'::uuid, 'eb0a594f-3a7e-4ca4-9d45-301826765b2a'::uuid, '550e8400-e29b-41d4-a716-446655440010'::uuid),
        ('0d02fc09-8b69-4ebd-bcb2-2da7456e49f1'::uuid, '078290d2-7429-47ff-8af3-25b48648bcb8'::uuid, 'a5eebc99-9c0b-4ef8-bb6d-6bb9bd380a99'::uuid, 'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616'::uuid, '550e8400-e29b-41d4-a716-446655440011'::uuid),
        ('0e988692-9b02-49f2-a389-de1213b93167'::uuid, '017516e4-a5ea-4244-8e2d-fa9c669c0695'::uuid, 'd4eebc99-9c0b-4ef8-bb6d-6bb9bd380a66'::uuid, '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d'::uuid, '550e8400-e29b-41d4-a716-446655440013'::uuid),
        ('0fa9d690-e4fa-4ae0-9b14-d6ba54249bd1'::uuid, 'fb68408d-c4d3-4d6e-b00f-0fdc1b01b14e'::uuid, 'b5eebc99-9c0b-4ef8-bb6d-6bb9bd380a00'::uuid, '709681ad-cde8-4af3-a986-642fec48e0e3'::uuid, '550e8400-e29b-41d4-a716-446655440010'::uuid),
        ('11e48741-5687-4b20-ad43-e88417e309d8'::uuid, 'b03d597d-908b-414b-9a0d-cad544ab06c2'::uuid, 'b6eebc99-9c0b-4ef8-bb6d-6bb9bd380a66'::uuid, '3facc0f8-0db8-4d5f-b01e-4393b78b2feb'::uuid, '550e8400-e29b-41d4-a716-446655440013'::uuid),
        ('18671034-b971-4480-b1be-6f9aad26f35a'::uuid, '67bef8f7-7e59-4bc4-8906-53e00819e6cd'::uuid, 'a1eebc99-9c0b-4ef8-bb6d-6bb9bd380a77'::uuid, '0c30e6a5-5e10-4a96-88f2-6f9823bc77f5'::uuid, '550e8400-e29b-41d4-a716-446655440010'::uuid),
        ('21375214-1aea-4642-bbb3-6e66d9774134'::uuid, '0caa3378-690b-4afd-bf03-c86a45cef878'::uuid, 'b6eebc99-9c0b-4ef8-bb6d-6bb9bd380a66'::uuid, '5cc348b9-236e-454e-97a7-5bdfad371be1'::uuid, '550e8400-e29b-41d4-a716-446655440012'::uuid)
    ) AS t(inv_id, var_id, prod_id, col_id, sz_id)
    JOIN inventory i ON i."id" = t.inv_id
    JOIN product_variants pv ON pv."id" = t.var_id
    JOIN products pr ON pr."id" = t.prod_id
    JOIN sizes sz ON sz."id" = t.sz_id
    JOIN colors cl ON cl."id" = t.col_id
),

-- 2. Random user lookup (any active user)
random_user AS (
    SELECT "id" AS user_id FROM users WHERE "deletedAt" IS NULL ORDER BY random() LIMIT 1
),

-- 3. Generate 10 orders with random statuses
order_definitions AS (
    SELECT
        generate_series(1, 10) AS order_index,
        (ARRAY['PENDING', 'CONFIRMED', 'PROCESSING', 'SHIPPED', 'DELIVERED', 'CANCELLED'])[floor(random() * 6 + 1)]::order_status AS order_status,
        (ARRAY['PENDING', 'COMPLETED', 'FAILED', 'PARTIALLY_REFUNDED'])[floor(random() * 4 + 1)]::payment_status AS payment_status
),

-- 4. For each order, generate 3 random inventory items and compute totals
order_items_data AS (
    SELECT
        od.order_index,
        ai.inventory_id,
        ai.sku,
        ai.product_name,
        ai.size_name,
        ai.color_name,
        ai.image_url,
        (floor(random() * 3 + 1))::int AS quantity,
        ai.unit_price,
        0 AS discount_amount
    FROM order_definitions od
    CROSS JOIN LATERAL (
        SELECT * FROM available_inventory ORDER BY random() LIMIT 3
    ) ai
),

order_totals AS (
    SELECT
        order_index,
        SUM(unit_price * quantity) AS subtotal,
        SUM(unit_price * quantity) * 0.05 AS tax_amount,
        CASE WHEN random() > 0.7 THEN 0 ELSE (floor(random() * 500 + 1))::decimal(10,2) END AS shipping_fee,
        SUM(unit_price * quantity) + (SUM(unit_price * quantity) * 0.05) +
            CASE WHEN random() > 0.7 THEN 0 ELSE (floor(random() * 500 + 1))::decimal(10,2) END AS total_amount
    FROM order_items_data
    GROUP BY order_index
),

-- 5. Insert orders (explicitly qualify order_index)
inserted_orders AS (
    INSERT INTO orders (
        "id",
        "orderNumber",
        "userId",
        "status",
        "paymentStatus",
        "subtotal",
        "discountAmount",
        "shippingFee",
        "taxAmount",
        "totalAmount",
        "notes",
        "cancelReason",
        "createdAt",
        "updatedAt"
    )
    SELECT
        gen_random_uuid(),
        'ORD-2026-' || lpad(od.order_index::text, 4, '0'),  -- use od.order_index
        (SELECT user_id FROM random_user),
        od.order_status,
        od.payment_status,
        ot.subtotal,
        0,
        ot.shipping_fee,
        ot.tax_amount,
        ot.total_amount,
        'Test order ' || od.order_index,
        CASE WHEN od.order_status = 'CANCELLED' THEN 'Cancelled by test' ELSE NULL END,
        NOW() - (random() * interval '30 days'),
        NOW() - (random() * interval '30 days')
    FROM order_definitions od
    JOIN order_totals ot ON od.order_index = ot.order_index
    RETURNING "id", "orderNumber"
),

-- attach order_index for mapping
order_index_map AS (
    SELECT
        io."id" AS order_id,
        (substring(io."orderNumber" from 'ORD-2026-(\d+)$')::int) AS order_index
    FROM inserted_orders io
),

-- 6. Insert order items (3 per order)
inserted_order_items AS (
    INSERT INTO order_items (
        "id",
        "orderId",
        "inventoryId",
        "sku",
        "productName",
        "sizeName",
        "colorName",
        "imageUrl",
        "quantity",
        "unitPrice",
        "discountAmount",
        "totalPrice",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        oid.inventory_id,
        oid.sku,
        oid.product_name,
        oid.size_name,
        oid.color_name,
        oid.image_url,
        oid.quantity,
        oid.unit_price,
        oid.discount_amount,
        oid.unit_price * oid.quantity,
        NOW() - (random() * interval '30 days')
    FROM order_index_map oim
    JOIN order_items_data oid ON oim.order_index = oid.order_index
    RETURNING "id"
),

-- 7. Insert order addresses
inserted_order_addresses AS (
    INSERT INTO order_addresses (
        "id",
        "orderId",
        "type",
        "houseNo",
        "addressLine1",
        "addressLine2",
        "city",
        "province",
        "zipcode",
        "country",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        'DELIVERY'::address_type,
        (floor(random() * 200 + 1))::text,
        'Street ' || floor(random() * 100)::text,
        CASE WHEN random() > 0.5 THEN 'Apt ' || floor(random() * 20)::text ELSE NULL END,
        (ARRAY['Colombo', 'Kandy', 'Galle', 'Jaffna'])[floor(random() * 4 + 1)],
        (ARRAY['Western', 'Central', 'Southern', 'Northern'])[floor(random() * 4 + 1)],
        (ARRAY['00100', '20000', '80000', '40000'])[floor(random() * 4 + 1)],
        'Sri Lanka',
        NOW() - (random() * interval '30 days')
    FROM order_index_map oim
    RETURNING "id"
),

-- 8. Insert order status history (initial status)
inserted_status_history AS (
    INSERT INTO order_status_history (
        "id",
        "orderId",
        "status",
        "notes",
        "createdBy",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        od.order_status,
        'Order created',
        NULL,
        NOW() - (random() * interval '30 days')
    FROM order_index_map oim
    JOIN order_definitions od ON oim.order_index = od.order_index
    RETURNING "id"
),

-- 9. Insert payments
inserted_payments AS (
    INSERT INTO payments (
        "id",
        "orderId",
        "amount",
        "currency",
        "method",
        "status",
        "gatewayProvider",
        "gatewayTransactionId",
        "gatewayResponse",
        "failureReason",
        "paidAt",
        "refundedAt",
        "createdAt",
        "updatedAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        ot.total_amount,
        'LKR',
        (ARRAY['CARD'::payment_method, 'BANK_TRANSFER', 'CASH_ON_DELIVERY', 'PAYHERE'])[floor(random() * 4 + 1)],
        od.payment_status,
        CASE WHEN od.payment_status IN ('COMPLETED','FAILED') THEN 'PayHere' ELSE NULL END,
        CASE WHEN od.payment_status IN ('COMPLETED','FAILED') THEN 'txn_' || gen_random_uuid()::text ELSE NULL END,
        '{"test": true}'::jsonb,
        CASE WHEN od.payment_status = 'FAILED' THEN 'Payment failure test' ELSE NULL END,
        CASE WHEN od.payment_status = 'COMPLETED' THEN NOW() - (random() * interval '10 days') ELSE NULL END,
        NULL,
        NOW() - (random() * interval '30 days'),
        NOW() - (random() * interval '30 days')
    FROM order_index_map oim
    JOIN order_totals ot ON oim.order_index = ot.order_index
    JOIN order_definitions od ON oim.order_index = od.order_index
    RETURNING "id"
)

-- Final summary
SELECT 'Orders inserted: 10' AS result
UNION ALL
SELECT 'Order items inserted: ' || COUNT(*)::text FROM inserted_order_items
UNION ALL
SELECT 'Order addresses inserted: ' || COUNT(*)::text FROM inserted_order_addresses
UNION ALL
SELECT 'Status history inserted: ' || COUNT(*)::text FROM inserted_status_history
UNION ALL
SELECT 'Payments inserted: ' || COUNT(*)::text FROM inserted_payments;
```
```postgresql
-- =====================================================
-- Insert 10 orders (status = PROCESSING), each with 3 order items,
-- address, status history, and payment
-- =====================================================

WITH
-- 1. List of available inventory items (same IDs as before)
available_inventory AS (
    SELECT
        i."id" AS inventory_id,
        pv."sku",
        pr."name" AS product_name,
        sz."name" AS size_name,
        cl."name" AS color_name,
        NULL::varchar AS image_url,
        pv."sellingPrice" AS unit_price
    FROM (VALUES
        ('03c9123e-e112-4583-95b6-8b1454590dbb'::uuid, '79119d27-bef1-4052-afef-6f220df90bf9'::uuid, 'b5eebc99-9c0b-4ef8-bb6d-6bb9bd380a00'::uuid, '4115386c-9ef6-4e68-928c-339392a4b708'::uuid, '550e8400-e29b-41d4-a716-446655440011'::uuid),
        ('046b324d-f7e3-40ce-8972-0b0b7db164d7'::uuid, '2ef665d0-78c5-46eb-8c3a-54fa4443d54f'::uuid, 'c2eebc99-9c0b-4ef8-bb6d-6bb9bd380a33'::uuid, '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d'::uuid, '550e8400-e29b-41d4-a716-446655440013'::uuid),
        ('078db6e4-b7b3-417b-9f39-ef4dc9b42c74'::uuid, '2af5a689-db40-4057-b83d-21575ff94818'::uuid, 'f5eebc99-9c0b-4ef8-bb6d-6bb9bd380a44'::uuid, '6662b302-0a77-42ff-a6ff-307f36f36151'::uuid, '550e8400-e29b-41d4-a716-446655440012'::uuid),
        ('0c7b2cf0-d2bd-4404-ba98-ed75e3be08fc'::uuid, '542e4b56-c5ea-46b5-8df3-aedaa09fdcdf'::uuid, 'b2eebc99-9c0b-4ef8-bb6d-6bb9bd380a22'::uuid, 'eb0a594f-3a7e-4ca4-9d45-301826765b2a'::uuid, '550e8400-e29b-41d4-a716-446655440010'::uuid),
        ('0d02fc09-8b69-4ebd-bcb2-2da7456e49f1'::uuid, '078290d2-7429-47ff-8af3-25b48648bcb8'::uuid, 'a5eebc99-9c0b-4ef8-bb6d-6bb9bd380a99'::uuid, 'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616'::uuid, '550e8400-e29b-41d4-a716-446655440011'::uuid),
        ('0e988692-9b02-49f2-a389-de1213b93167'::uuid, '017516e4-a5ea-4244-8e2d-fa9c669c0695'::uuid, 'd4eebc99-9c0b-4ef8-bb6d-6bb9bd380a66'::uuid, '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d'::uuid, '550e8400-e29b-41d4-a716-446655440013'::uuid),
        ('0fa9d690-e4fa-4ae0-9b14-d6ba54249bd1'::uuid, 'fb68408d-c4d3-4d6e-b00f-0fdc1b01b14e'::uuid, 'b5eebc99-9c0b-4ef8-bb6d-6bb9bd380a00'::uuid, '709681ad-cde8-4af3-a986-642fec48e0e3'::uuid, '550e8400-e29b-41d4-a716-446655440010'::uuid),
        ('11e48741-5687-4b20-ad43-e88417e309d8'::uuid, 'b03d597d-908b-414b-9a0d-cad544ab06c2'::uuid, 'b6eebc99-9c0b-4ef8-bb6d-6bb9bd380a66'::uuid, '3facc0f8-0db8-4d5f-b01e-4393b78b2feb'::uuid, '550e8400-e29b-41d4-a716-446655440013'::uuid),
        ('18671034-b971-4480-b1be-6f9aad26f35a'::uuid, '67bef8f7-7e59-4bc4-8906-53e00819e6cd'::uuid, 'a1eebc99-9c0b-4ef8-bb6d-6bb9bd380a77'::uuid, '0c30e6a5-5e10-4a96-88f2-6f9823bc77f5'::uuid, '550e8400-e29b-41d4-a716-446655440010'::uuid),
        ('21375214-1aea-4642-bbb3-6e66d9774134'::uuid, '0caa3378-690b-4afd-bf03-c86a45cef878'::uuid, 'b6eebc99-9c0b-4ef8-bb6d-6bb9bd380a66'::uuid, '5cc348b9-236e-454e-97a7-5bdfad371be1'::uuid, '550e8400-e29b-41d4-a716-446655440012'::uuid)
    ) AS t(inv_id, var_id, prod_id, col_id, sz_id)
    JOIN inventory i ON i."id" = t.inv_id
    JOIN product_variants pv ON pv."id" = t.var_id
    JOIN products pr ON pr."id" = t.prod_id
    JOIN sizes sz ON sz."id" = t.sz_id
    JOIN colors cl ON cl."id" = t.col_id
),

-- 2. Random user lookup (any active user)
random_user AS (
    SELECT "id" AS user_id FROM users WHERE "deletedAt" IS NULL ORDER BY random() LIMIT 1
),

-- 3. Generate 10 orders with PROCESSING status and random payment status
order_definitions AS (
    SELECT
        generate_series(11, 20) AS order_index,   -- start from 11 to avoid conflict
        'PROCESSING'::order_status AS order_status,
        (ARRAY['PENDING', 'COMPLETED', 'FAILED', 'PARTIALLY_REFUNDED'])[floor(random() * 4 + 1)]::payment_status AS payment_status
),

-- 4. For each order, generate 3 random inventory items and compute totals
order_items_data AS (
    SELECT
        od.order_index,
        ai.inventory_id,
        ai.sku,
        ai.product_name,
        ai.size_name,
        ai.color_name,
        ai.image_url,
        (floor(random() * 3 + 1))::int AS quantity,
        ai.unit_price,
        0 AS discount_amount
    FROM order_definitions od
    CROSS JOIN LATERAL (
        SELECT * FROM available_inventory ORDER BY random() LIMIT 3
    ) ai
),

order_totals AS (
    SELECT
        order_index,
        SUM(unit_price * quantity) AS subtotal,
        SUM(unit_price * quantity) * 0.05 AS tax_amount,
        CASE WHEN random() > 0.7 THEN 0 ELSE (floor(random() * 500 + 1))::decimal(10,2) END AS shipping_fee,
        SUM(unit_price * quantity) + (SUM(unit_price * quantity) * 0.05) +
            CASE WHEN random() > 0.7 THEN 0 ELSE (floor(random() * 500 + 1))::decimal(10,2) END AS total_amount
    FROM order_items_data
    GROUP BY order_index
),

-- 5. Insert orders (status = PROCESSING)
inserted_orders AS (
    INSERT INTO orders (
        "id",
        "orderNumber",
        "userId",
        "status",
        "paymentStatus",
        "subtotal",
        "discountAmount",
        "shippingFee",
        "taxAmount",
        "totalAmount",
        "notes",
        "cancelReason",
        "createdAt",
        "updatedAt"
    )
    SELECT
        gen_random_uuid(),
        'ORD-2026-PROC-' || lpad(od.order_index::text, 4, '0'),
        (SELECT user_id FROM random_user),
        od.order_status,
        od.payment_status,
        ot.subtotal,
        0,
        ot.shipping_fee,
        ot.tax_amount,
        ot.total_amount,
        'Processing order ' || od.order_index,
        NULL,
        NOW() - (random() * interval '30 days'),
        NOW() - (random() * interval '30 days')
    FROM order_definitions od
    JOIN order_totals ot ON od.order_index = ot.order_index
    RETURNING "id", "orderNumber"
),

-- attach order_index for mapping
order_index_map AS (
    SELECT
        io."id" AS order_id,
        (substring(io."orderNumber" from 'ORD-2026-PROC-(\d+)$')::int) AS order_index
    FROM inserted_orders io
),

-- 6. Insert order items (3 per order)
inserted_order_items AS (
    INSERT INTO order_items (
        "id",
        "orderId",
        "inventoryId",
        "sku",
        "productName",
        "sizeName",
        "colorName",
        "imageUrl",
        "quantity",
        "unitPrice",
        "discountAmount",
        "totalPrice",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        oid.inventory_id,
        oid.sku,
        oid.product_name,
        oid.size_name,
        oid.color_name,
        oid.image_url,
        oid.quantity,
        oid.unit_price,
        oid.discount_amount,
        oid.unit_price * oid.quantity,
        NOW() - (random() * interval '30 days')
    FROM order_index_map oim
    JOIN order_items_data oid ON oim.order_index = oid.order_index
    RETURNING "id"
),

-- 7. Insert order addresses
inserted_order_addresses AS (
    INSERT INTO order_addresses (
        "id",
        "orderId",
        "type",
        "houseNo",
        "addressLine1",
        "addressLine2",
        "city",
        "province",
        "zipcode",
        "country",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        'DELIVERY'::address_type,
        (floor(random() * 200 + 1))::text,
        'Street ' || floor(random() * 100)::text,
        CASE WHEN random() > 0.5 THEN 'Apt ' || floor(random() * 20)::text ELSE NULL END,
        (ARRAY['Colombo', 'Kandy', 'Galle', 'Jaffna'])[floor(random() * 4 + 1)],
        (ARRAY['Western', 'Central', 'Southern', 'Northern'])[floor(random() * 4 + 1)],
        (ARRAY['00100', '20000', '80000', '40000'])[floor(random() * 4 + 1)],
        'Sri Lanka',
        NOW() - (random() * interval '30 days')
    FROM order_index_map oim
    RETURNING "id"
),

-- 8. Insert order status history (initial status = PROCESSING)
inserted_status_history AS (
    INSERT INTO order_status_history (
        "id",
        "orderId",
        "status",
        "notes",
        "createdBy",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        'PROCESSING'::order_status,
        'Order is being processed',
        NULL,
        NOW() - (random() * interval '30 days')
    FROM order_index_map oim
    RETURNING "id"
),

-- 9. Insert payments
inserted_payments AS (
    INSERT INTO payments (
        "id",
        "orderId",
        "amount",
        "currency",
        "method",
        "status",
        "gatewayProvider",
        "gatewayTransactionId",
        "gatewayResponse",
        "failureReason",
        "paidAt",
        "refundedAt",
        "createdAt",
        "updatedAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        ot.total_amount,
        'LKR',
        (ARRAY['CARD'::payment_method, 'BANK_TRANSFER', 'CASH_ON_DELIVERY', 'PAYHERE'])[floor(random() * 4 + 1)],
        od.payment_status,
        CASE WHEN od.payment_status IN ('COMPLETED','FAILED') THEN 'PayHere' ELSE NULL END,
        CASE WHEN od.payment_status IN ('COMPLETED','FAILED') THEN 'txn_' || gen_random_uuid()::text ELSE NULL END,
        '{"test": true}'::jsonb,
        CASE WHEN od.payment_status = 'FAILED' THEN 'Payment failure test' ELSE NULL END,
        CASE WHEN od.payment_status = 'COMPLETED' THEN NOW() - (random() * interval '10 days') ELSE NULL END,
        NULL,
        NOW() - (random() * interval '30 days'),
        NOW() - (random() * interval '30 days')
    FROM order_index_map oim
    JOIN order_totals ot ON oim.order_index = ot.order_index
    JOIN order_definitions od ON oim.order_index = od.order_index
    RETURNING "id"
)

-- Final summary
SELECT 'Orders inserted (PROCESSING): 10' AS result
UNION ALL
SELECT 'Order items inserted: ' || COUNT(*)::text FROM inserted_order_items
UNION ALL
SELECT 'Order addresses inserted: ' || COUNT(*)::text FROM inserted_order_addresses
UNION ALL
SELECT 'Status history inserted: ' || COUNT(*)::text FROM inserted_status_history
UNION ALL
SELECT 'Payments inserted: ' || COUNT(*)::text FROM inserted_payments;
```
```postgresql
-- =====================================================
-- Insert 20 orders, each with 2 order items,
-- random status from: PENDING, CONFIRMED, PROCESSING,
-- SHIPPED, DELIVERED, CANCELLED
-- =====================================================

WITH
-- 1. Available inventory items (same set as before)
available_inventory AS (
    SELECT
        i."id" AS inventory_id,
        pv."sku",
        pr."name" AS product_name,
        sz."name" AS size_name,
        cl."name" AS color_name,
        NULL::varchar AS image_url,
        pv."sellingPrice" AS unit_price
    FROM (VALUES
        ('03c9123e-e112-4583-95b6-8b1454590dbb'::uuid, '79119d27-bef1-4052-afef-6f220df90bf9'::uuid, 'b5eebc99-9c0b-4ef8-bb6d-6bb9bd380a00'::uuid, '4115386c-9ef6-4e68-928c-339392a4b708'::uuid, '550e8400-e29b-41d4-a716-446655440011'::uuid),
        ('046b324d-f7e3-40ce-8972-0b0b7db164d7'::uuid, '2ef665d0-78c5-46eb-8c3a-54fa4443d54f'::uuid, 'c2eebc99-9c0b-4ef8-bb6d-6bb9bd380a33'::uuid, '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d'::uuid, '550e8400-e29b-41d4-a716-446655440013'::uuid),
        ('078db6e4-b7b3-417b-9f39-ef4dc9b42c74'::uuid, '2af5a689-db40-4057-b83d-21575ff94818'::uuid, 'f5eebc99-9c0b-4ef8-bb6d-6bb9bd380a44'::uuid, '6662b302-0a77-42ff-a6ff-307f36f36151'::uuid, '550e8400-e29b-41d4-a716-446655440012'::uuid),
        ('0c7b2cf0-d2bd-4404-ba98-ed75e3be08fc'::uuid, '542e4b56-c5ea-46b5-8df3-aedaa09fdcdf'::uuid, 'b2eebc99-9c0b-4ef8-bb6d-6bb9bd380a22'::uuid, 'eb0a594f-3a7e-4ca4-9d45-301826765b2a'::uuid, '550e8400-e29b-41d4-a716-446655440010'::uuid),
        ('0d02fc09-8b69-4ebd-bcb2-2da7456e49f1'::uuid, '078290d2-7429-47ff-8af3-25b48648bcb8'::uuid, 'a5eebc99-9c0b-4ef8-bb6d-6bb9bd380a99'::uuid, 'a3dbbeea-ba0a-4dea-bfa4-ab0cdfcb2616'::uuid, '550e8400-e29b-41d4-a716-446655440011'::uuid),
        ('0e988692-9b02-49f2-a389-de1213b93167'::uuid, '017516e4-a5ea-4244-8e2d-fa9c669c0695'::uuid, 'd4eebc99-9c0b-4ef8-bb6d-6bb9bd380a66'::uuid, '019e4ca6-fd4a-4ae7-ad58-9dd68d8d0e5d'::uuid, '550e8400-e29b-41d4-a716-446655440013'::uuid),
        ('0fa9d690-e4fa-4ae0-9b14-d6ba54249bd1'::uuid, 'fb68408d-c4d3-4d6e-b00f-0fdc1b01b14e'::uuid, 'b5eebc99-9c0b-4ef8-bb6d-6bb9bd380a00'::uuid, '709681ad-cde8-4af3-a986-642fec48e0e3'::uuid, '550e8400-e29b-41d4-a716-446655440010'::uuid),
        ('11e48741-5687-4b20-ad43-e88417e309d8'::uuid, 'b03d597d-908b-414b-9a0d-cad544ab06c2'::uuid, 'b6eebc99-9c0b-4ef8-bb6d-6bb9bd380a66'::uuid, '3facc0f8-0db8-4d5f-b01e-4393b78b2feb'::uuid, '550e8400-e29b-41d4-a716-446655440013'::uuid),
        ('18671034-b971-4480-b1be-6f9aad26f35a'::uuid, '67bef8f7-7e59-4bc4-8906-53e00819e6cd'::uuid, 'a1eebc99-9c0b-4ef8-bb6d-6bb9bd380a77'::uuid, '0c30e6a5-5e10-4a96-88f2-6f9823bc77f5'::uuid, '550e8400-e29b-41d4-a716-446655440010'::uuid),
        ('21375214-1aea-4642-bbb3-6e66d9774134'::uuid, '0caa3378-690b-4afd-bf03-c86a45cef878'::uuid, 'b6eebc99-9c0b-4ef8-bb6d-6bb9bd380a66'::uuid, '5cc348b9-236e-454e-97a7-5bdfad371be1'::uuid, '550e8400-e29b-41d4-a716-446655440012'::uuid)
    ) AS t(inv_id, var_id, prod_id, col_id, sz_id)
    JOIN inventory i ON i."id" = t.inv_id
    JOIN product_variants pv ON pv."id" = t.var_id
    JOIN products pr ON pr."id" = t.prod_id
    JOIN sizes sz ON sz."id" = t.sz_id
    JOIN colors cl ON cl."id" = t.col_id
),

-- 2. Random user
random_user AS (
    SELECT "id" AS user_id FROM users WHERE "deletedAt" IS NULL ORDER BY random() LIMIT 1
),

-- 3. Generate 20 orders with random status from the allowed list
order_definitions AS (
    SELECT
        generate_series(31, 50) AS order_index,
        (ARRAY['PENDING', 'CONFIRMED', 'PROCESSING', 'SHIPPED', 'DELIVERED', 'CANCELLED'])[floor(random() * 6 + 1)]::order_status AS order_status,
        (ARRAY['PENDING', 'COMPLETED', 'FAILED', 'PARTIALLY_REFUNDED'])[floor(random() * 4 + 1)]::payment_status AS payment_status
),

-- 4. For each order, generate 2 random inventory items
order_items_data AS (
    SELECT
        od.order_index,
        ai.inventory_id,
        ai.sku,
        ai.product_name,
        ai.size_name,
        ai.color_name,
        ai.image_url,
        (floor(random() * 3 + 1))::int AS quantity,   -- quantity 1-3
        ai.unit_price,
        0 AS discount_amount
    FROM order_definitions od
    CROSS JOIN LATERAL (
        SELECT * FROM available_inventory ORDER BY random() LIMIT 2   -- exactly 2 items
    ) ai
),

order_totals AS (
    SELECT
        order_index,
        SUM(unit_price * quantity) AS subtotal,
        SUM(unit_price * quantity) * 0.05 AS tax_amount,
        CASE WHEN random() > 0.7 THEN 0 ELSE (floor(random() * 500 + 1))::decimal(10,2) END AS shipping_fee,
        SUM(unit_price * quantity) + (SUM(unit_price * quantity) * 0.05) +
            CASE WHEN random() > 0.7 THEN 0 ELSE (floor(random() * 500 + 1))::decimal(10,2) END AS total_amount
    FROM order_items_data
    GROUP BY order_index
),

-- 5. Insert orders
inserted_orders AS (
    INSERT INTO orders (
        "id",
        "orderNumber",
        "userId",
        "status",
        "paymentStatus",
        "subtotal",
        "discountAmount",
        "shippingFee",
        "taxAmount",
        "totalAmount",
        "notes",
        "cancelReason",
        "createdAt",
        "updatedAt"
    )
    SELECT
        gen_random_uuid(),
        'ORD-2026-RND-' || lpad(od.order_index::text, 4, '0'),
        (SELECT user_id FROM random_user),
        od.order_status,
        od.payment_status,
        ot.subtotal,
        0,
        ot.shipping_fee,
        ot.tax_amount,
        ot.total_amount,
        'Random status order ' || od.order_index,
        CASE WHEN od.order_status = 'CANCELLED' THEN 'Cancelled by system test' ELSE NULL END,
        NOW() - (random() * interval '90 days'),
        NOW() - (random() * interval '60 days')
    FROM order_definitions od
    JOIN order_totals ot ON od.order_index = ot.order_index
    RETURNING "id", "orderNumber"
),

-- attach order_index for mapping
order_index_map AS (
    SELECT
        io."id" AS order_id,
        (substring(io."orderNumber" from 'ORD-2026-RND-(\d+)$')::int) AS order_index
    FROM inserted_orders io
),

-- 6. Insert order items (2 per order)
inserted_order_items AS (
    INSERT INTO order_items (
        "id",
        "orderId",
        "inventoryId",
        "sku",
        "productName",
        "sizeName",
        "colorName",
        "imageUrl",
        "quantity",
        "unitPrice",
        "discountAmount",
        "totalPrice",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        oid.inventory_id,
        oid.sku,
        oid.product_name,
        oid.size_name,
        oid.color_name,
        oid.image_url,
        oid.quantity,
        oid.unit_price,
        oid.discount_amount,
        oid.unit_price * oid.quantity,
        NOW() - (random() * interval '90 days')
    FROM order_index_map oim
    JOIN order_items_data oid ON oim.order_index = oid.order_index
    RETURNING "id"
),

-- 7. Insert order addresses
inserted_order_addresses AS (
    INSERT INTO order_addresses (
        "id",
        "orderId",
        "type",
        "houseNo",
        "addressLine1",
        "addressLine2",
        "city",
        "province",
        "zipcode",
        "country",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        'DELIVERY'::address_type,
        (floor(random() * 200 + 1))::text,
        'Street ' || floor(random() * 100)::text,
        CASE WHEN random() > 0.5 THEN 'Apt ' || floor(random() * 20)::text ELSE NULL END,
        (ARRAY['Colombo', 'Kandy', 'Galle', 'Jaffna'])[floor(random() * 4 + 1)],
        (ARRAY['Western', 'Central', 'Southern', 'Northern'])[floor(random() * 4 + 1)],
        (ARRAY['00100', '20000', '80000', '40000'])[floor(random() * 4 + 1)],
        'Sri Lanka',
        NOW() - (random() * interval '90 days')
    FROM order_index_map oim
    RETURNING "id"
),

-- 8. Insert order status history (initial status)
inserted_status_history AS (
    INSERT INTO order_status_history (
        "id",
        "orderId",
        "status",
        "notes",
        "createdBy",
        "createdAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        od.order_status,
        'Order created with initial status: ' || od.order_status,
        NULL,
        NOW() - (random() * interval '90 days')
    FROM order_index_map oim
    JOIN order_definitions od ON oim.order_index = od.order_index
    RETURNING "id"
),

-- 9. Insert payments
inserted_payments AS (
    INSERT INTO payments (
        "id",
        "orderId",
        "amount",
        "currency",
        "method",
        "status",
        "gatewayProvider",
        "gatewayTransactionId",
        "gatewayResponse",
        "failureReason",
        "paidAt",
        "refundedAt",
        "createdAt",
        "updatedAt"
    )
    SELECT
        gen_random_uuid(),
        oim.order_id,
        ot.total_amount,
        'LKR',
        (ARRAY['CARD'::payment_method, 'BANK_TRANSFER', 'CASH_ON_DELIVERY', 'PAYHERE'])[floor(random() * 4 + 1)],
        od.payment_status,
        CASE WHEN od.payment_status IN ('COMPLETED','FAILED') THEN 'PayHere' ELSE NULL END,
        CASE WHEN od.payment_status IN ('COMPLETED','FAILED') THEN 'txn_' || gen_random_uuid()::text ELSE NULL END,
        '{"test": true}'::jsonb,
        CASE WHEN od.payment_status = 'FAILED' THEN 'Payment failure' ELSE NULL END,
        CASE WHEN od.payment_status = 'COMPLETED' THEN NOW() - (random() * interval '80 days') ELSE NULL END,
        NULL,
        NOW() - (random() * interval '90 days'),
        NOW() - (random() * interval '60 days')
    FROM order_index_map oim
    JOIN order_totals ot ON oim.order_index = ot.order_index
    JOIN order_definitions od ON oim.order_index = od.order_index
    RETURNING "id"
)

-- Final summary
SELECT 'Orders inserted (random status, 20): ' || COUNT(*)::text FROM inserted_orders
UNION ALL
SELECT 'Order items inserted: ' || COUNT(*)::text FROM inserted_order_items
UNION ALL
SELECT 'Order addresses inserted: ' || COUNT(*)::text FROM inserted_order_addresses
UNION ALL
SELECT 'Status history inserted: ' || COUNT(*)::text FROM inserted_status_history
UNION ALL
SELECT 'Payments inserted: ' || COUNT(*)::text FROM inserted_payments;
```