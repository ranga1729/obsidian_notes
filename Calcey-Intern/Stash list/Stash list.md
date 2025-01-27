Order form task
branch: order-form-new
(only front end)

SELECT feedback form - 2nd task
branch: recipe-sorting
(only front end)

SELECT feedback form - 9th task(backend) + add a button to select menu items to filter select meal variations created on that meal code.(front end)
branch: meal-code-search
created a merge request for front end in 1st OCT 2024.
back end part it yet to be committed. 

![[Pasted image 20240929125241.png]]


remove sortOrder and calculate it inside the sorting function.
set a PR to that. 
set a PR to back end task.

take meal-management's meal-variation's meal code and create meal-variations in select tab. 1 meal variation for 1 meal code.
portion size: medium
dietary preference: standard



late checkout on 24 ?
half board for 22, price ?
26,000 - 

22, 23, 24 

call Pelmadulla(Terrace cinamon view) - ask for a bathing place ?
rooms availability on 22, 23, 24 for 17: yes 18 : 90,000 per night + chef
full board packages: 
half board packages: 
bathing places: 

```
select "Ingredients"."Name" as "Ingredients.Name", "Ingredients"."Allergen" as "Ingredients.Allergen", "IngredientGroups"."Name" as "IngredientGroups.Name", "IngredientGroups"."Allergen" as "IngredientGroups.Allergen" from "Ingredients" 
	join "IngredientGroupMappings" on "Ingredients"."Id" = "IngredientGroupMappings"."IngredientId"
	join "IngredientGroups" on "IngredientGroups"."Id" = "IngredientGroupMappings"."GroupId"
 where "IngredientGroups"."Name" = 'Sulphites'
```
