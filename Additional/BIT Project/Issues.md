Prisma client was configured to filter-out deleted records automatically from all the queries to implement soft-delete.
This created conflicts when, if there is a record deleted(soft-deleted) (let's say "Red"), and we are trying to update an non-deleted active records name("Red-2") into that soft-deleted record's name. 

Solution: First delete the soft-deleted record first, then update the current record's name into the requested name. 

But the prisma client was configured to filter out soft-deleted record even from the `delete` and `deleteMany` queries. 

Applied solution: rolled back the prisma client into it's original form. Handling soft-deleting manually. 