<script>
  import { writable } from "svelte/store";
  import { onMount, onDestroy } from "svelte";

  const ingredients = writable([
    { name: "", cost: "", volumePack: "", volumeRecipe: "" },
  ]);

  let ingredient = { name: "", cost: "", volumePack: "", volumeRecipe: "" };
// Load ingredients from localStorage when the component mounts
onMount(() => {
  if (typeof window !== "undefined") {
    const storedIngredients = JSON.parse(localStorage.getItem("ingredients") || "[]");
    ingredients.set(storedIngredients);
  }
});


  // Save ingredients to localStorage when the component is destroyed
  onDestroy(() => {
    if (typeof window !== "undefined" && window.localStorage) {
      ingredients.subscribe((value) => {
        localStorage.setItem("ingredients", JSON.stringify(value));
      });
    }
  });

  // Add a new ingredient to the list and update localStorage
  function addIngredient() {
    ingredients.update((arr) => {
      const newArr = [...arr, { ...ingredient }];
      localStorage.setItem("ingredients", JSON.stringify(newArr));
      return newArr;
    });
  }

  // Clear all ingredients from the list and update localStorage
  function clearIngredients() {
    ingredients.set([{ name: "", cost: "", volumePack: "", volumeRecipe: "" }]);
    localStorage.setItem("ingredients", JSON.stringify([{ name: "", cost: "", volumePack: "", volumeRecipe: "" }]));
  }

  // Update localStorage whenever ingredients change
  function updateLocalStorage() {
    ingredients.subscribe((value) => {
      localStorage.setItem("ingredients", JSON.stringify(value));
    });
  }

  // Remove an ingredient from the list by index and update localStorage
  /**
   * @param {number} index - The index of the ingredient to remove.
   */
  function removeIngredient(index) {
    ingredients.update((arr) => {
      const newArr = [...arr];
      newArr.splice(index, 1);
      localStorage.setItem("ingredients", JSON.stringify(newArr));
      return newArr;
    });
  }

  /**
   * Calculate the total cost of all ingredients
   * @param {{name: string, cost: string, volumePack: string, volumeRecipe: string}[]} ingredients
   * @returns {number}
   */
  function calculateTotalCost(ingredients) {
    return ingredients.reduce((acc, curr) => {
      const { cost, volumePack, volumeRecipe } = curr;
      const costNum = parseFloat(cost);
      const volumePackNum = parseFloat(volumePack);
      const volumeRecipeNum = parseFloat(volumeRecipe);

      if (!isNaN(costNum) && !isNaN(volumePackNum) && !isNaN(volumeRecipeNum)) {
        const ingredientCost = costNum * (volumeRecipeNum / volumePackNum);
        return acc + ingredientCost;
      }
      return acc;
    }, 0);
  }
</script>

<style>
  .table td, .table th {
    padding: 0.25rem;
    text-align: left;
    white-space: normal; /* Позволяет тексту переноситься на новые строки */
    word-wrap: break-word; /* Переносит слова при необходимости */
  }

  .table input {
    width: 100%;
    box-sizing: border-box;
    padding: 0.5rem;
  }
</style>

<!-- table -->
<div class="overflow-x-auto">
  <table class="table">

    <!-- head -->
    <thead>
      <tr>
        <th>#</th>
        <th>название ингредиента</th>
        <th>стоимость упаковки <span class="font-normal italic">(₽)</span></th>
        <th>объем в&nbsp;упаковке <span class="font-normal text-nowrap italic">(мл/г/шт)</span></th>
        <th>объем в&nbsp;рецепте <span class="font-normal text-nowrap italic">(мл/г/шт)</span></th>
        <th></th>
      </tr>
    </thead>

    <!-- body -->
    <tbody>
      {#each $ingredients as ingredient, index}
      <tr>
        <th class="text-xs bg-base-200 text-neutral">{index + 1}</th>
        <td><input type="text" class="input input-bordered border-dashed" on:input={updateLocalStorage} bind:value={ingredient.name} /></td>
        <td><input type="number" class="input input-bordered border-dashed" on:input={updateLocalStorage} bind:value={ingredient.cost} /></td>
        <td><input type="number" class="input input-bordered border-dashed" on:input={updateLocalStorage} bind:value={ingredient.volumePack} /></td>
        <td><input type="number" class="input input-bordered border-dashed" on:input={updateLocalStorage} bind:value={ingredient.volumeRecipe} /></td>
        <td>
          <button on:click={() => removeIngredient(index)}>❌</button>
        </td>
      </tr>
      {/each}
    </tbody>
  </table>
</div>



<!-- buttons -->
<div class="flex flex-col items-center space-y-4 p-4">
  <button class="btn" on:click={addIngredient}>добавить ингредиент</button>

  <button class="btn" on:click={clearIngredients}>очистить всё</button>

  <p class="flex items-center space-x-2">
    Себестоимость:
    <input class="input input-bordered w-full max-w-xs ml-2"
      disabled
      value="{$ingredients.length > 0
      ? calculateTotalCost($ingredients).toFixed(2)
      : '0.00'} ₽"/>
  </p>
</div>
