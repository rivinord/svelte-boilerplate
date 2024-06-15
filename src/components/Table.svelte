<script>
    import { writable } from "svelte/store";
    import { onMount, onDestroy } from "svelte";

    // Load ingredients from localStorage when the component mounts
    onMount(() => {
      if (typeof window !== "undefined" && window.localStorage) {
        const storedIngredients = JSON.parse(
          localStorage.getItem("ingredients") || "[]",
        );
        ingredients.set(storedIngredients);
      }
    });

    // Save ingredients to localStorage when the component is destroyed
    onDestroy(() => {
      if (typeof window !== "undefined" && window.localStorage) {
        localStorage.setItem(
          "ingredients",
          JSON.stringify(ingredients.subscribe((value) => value)),
        );
      }
    });

    /** @type {import('svelte/store').Writable<{name: string, cost: string, volumePack: string, volumeRecipe: string}[]>} */
    const ingredients = writable([
      { name: "", cost: "", volumePack: "", volumeRecipe: "" },
    ]);

    /** @type {number} */
    let total = 0;

    /** @type {{name: string, cost: string, volumePack: string, volumeRecipe: string}} */
    let ingredient = { name: "", cost: "", volumePack: "", volumeRecipe: "" };

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
      ingredients.set([]);
      localStorage.setItem("ingredients", JSON.stringify([]));
    }

    // Update localStorage whenever ingredients change
    function updateLocalStorage() {
      localStorage.setItem("ingredients", JSON.stringify($ingredients));
    }

    /**
     * Remove an ingredient from the list by index and update localStorage
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
          const ingredientCost = (costNum * volumeRecipeNum) / volumePackNum;
          return acc + ingredientCost;
        }
        return acc;
      }, 0);
    }
</script>

<style>
    .table td, .table th {
      padding: 0.5rem;
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
  
  <div class="overflow-x-auto">
      <table class="table">
        <!-- head -->
        <thead>
          <tr>
            <th>#</th>
            <th>Название ингредиента</th>
            <th>Цена упаковки</th>
            <th>Объем в&nbsp;упаковке</th>
            <th>Объем в&nbsp;рецепте</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          {#each $ingredients as ingredient, index}
          <tr>
            <th>{index + 1}</th>
            <td><input type="text" placeholder="Type here" class="input-ghost" on:input={updateLocalStorage} bind:value={ingredient.name} /></td>
            <td><input type="number" placeholder="Type here" class="input-ghost" on:input={updateLocalStorage} bind:value={ingredient.cost} /></td>
            <td><input type="number" placeholder="Type here" class="input-ghost" on:input={updateLocalStorage} bind:value={ingredient.volumePack} /></td>
            <td><input type="number" placeholder="Type here" class="input-ghost" on:input={updateLocalStorage} bind:value={ingredient.volumeRecipe} /></td>
            <td>
              <button on:click={() => removeIngredient(index)}>❌</button>
            </td>
          </tr>
          {/each}
        </tbody>
      </table>
  </div>
  
  <div>
      <button class="btn" on:click={addIngredient}>Add Ingredient</button>
  </div>
  
  <div>
      <button class="btn" on:click={clearIngredients}>Clear All</button>
  </div>
  
  <div>
      <p style="display: flex; align-items: center;">Себестоимость:
          <input class="input input-bordered w-full max-w-xs"
              disabled
              value="{$ingredients.length > 0
              ? calculateTotalCost($ingredients).toFixed(2)
              : '0.00'} руб"/>
      </p>
  </div>
  