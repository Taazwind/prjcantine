<template>
  <div id="app">
    <h1>Cantine scolaire</h1>

    <DayButtons :days="days" @select-day="selectDay" />

    <DishForm @add-dish="addDish" />

    <button id="api-btn" @click="importDishesFromApi" :disabled="isLoading">
      {{ isLoading ? "Chargement..." : "Importer des plats depuis l'API" }}
    </button>

    <div>
      <p>Nombre total de plats ajoutés : {{ totalDishes }}</p>
      <DishList
        :selectedDay="selectedDay"
        :customMenu="customMenu"
        @delete-dish="deleteDish"
      />
    </div>
  </div>
</template>

<script>
import { ref, reactive, computed } from "vue";
import DayButtons from "./components/DayButtons.vue";
import DishForm from "./components/DishForm.vue";
import DishList from "./components/DishList.vue";

export default {
  components: {
    DayButtons,
    DishForm,
    DishList,
  },
  setup() {
    const days = ["Lundi", "Mardi", "Mercredi", "Jeudi", "Vendredi"];
    const selectedDay = ref("Lundi");
    const isLoading = ref(false);

    const customMenu = reactive({});
    days.forEach((day) => {
      customMenu[day] = {
        Entrée: [],
        Plat: [],
        Dessert: [],
      };
    });

    const selectDay = (day) => {
      selectedDay.value = day;
    };

    const addDish = (newDish) => {
      if (newDish.title && newDish.description) {
        customMenu[selectedDay.value][newDish.category].push({
          ...newDish,
          votes: 0,
        });
      }
    };

    const deleteDish = (category, dish) => {
      const index = customMenu[selectedDay.value][category].indexOf(dish);
      if (index !== -1) {
        customMenu[selectedDay.value][category].splice(index, 1);
      }
    };

    const importDishesFromApi = async () => {
      isLoading.value = true;
      try {
        const response = await fetch(
          "https://tasty.p.rapidapi.com/recipes/list?from=0&size=100",
          {
            method: "GET",
            headers: {
              "X-RapidAPI-Key":
                "6050ac7df2msh45e75a2d5f5082bp137a59jsnce614ff252da",
              "X-RapidAPI-Host": "tasty.p.rapidapi.com",
            },
          }
        );

        if (!response.ok) {
          throw new Error(`Erreur HTTP : ${response.status}`);
        }

        const data = await response.json();

        // Mélanger les recettes de manière aléatoire
        const shuffledRecipes = shuffleArray(data.results);

        const starter = findUniqueDish(shuffledRecipes, "Entrée");
        const mainDish = findUniqueDish(shuffledRecipes, "Plat");
        const dessert = findUniqueDish(shuffledRecipes, "Dessert");

        if (starter && mainDish && dessert) {
          customMenu[selectedDay.value]["Entrée"].push({
            title: starter.name,
            description: starter.description || "Pas de description disponible",
            votes: 0,
          });

          customMenu[selectedDay.value]["Plat"].push({
            title: mainDish.name,
            description:
              mainDish.description || "Pas de description disponible",
            votes: 0,
          });

          customMenu[selectedDay.value]["Dessert"].push({
            title: dessert.name,
            description: dessert.description || "Pas de description disponible",
            votes: 0,
          });
        } else {
          console.error(
            "Impossible de trouver une entrée, un plat ou un dessert."
          );
        }
      } catch (error) {
        console.error("Erreur lors de l'importation :", error);
      } finally {
        isLoading.value = false;
      }
    };

    // Fonction de recherche d'un plat unique qui n'a pas déjà été affiché
    const findUniqueDish = (recipes, category) => {
      const dishFinder =
        category === "Entrée"
          ? isStarter
          : category === "Dessert"
          ? isDessert
          : category === "Plat"
          ? isMain
          : () => true;

      for (let recipe of recipes) {
        if (dishFinder(recipe)) {
          return recipe;
        }
      }
      return null;
    };

    // Fonction de mélange aléatoire
    const shuffleArray = (array) => {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    };

    const isDessert = (recipe) => {
      const keywords = [
        "cake",
        "ice cream",
        "dessert",
        "chocolate",
        "muffins",
        "macarons",
        "pancakes",
      ];
      return keywords.some((word) => recipe.name.toLowerCase().includes(word));
    };

    const isStarter = (recipe) => {
      const keywords = ["salad", "soup"];
      return keywords.some((word) => recipe.name.toLowerCase().includes(word));
    };

    const isMain = (recipe) => {
      const keywords = [
        "chicken",
        "beef",
        "steak",
        "fish",
        "pasta",
        "burger",
        "vegetables",
        "rice",
        "quinoa",
      ];
      return keywords.some((word) => recipe.name.toLowerCase().includes(word));
    };

    const totalDishes = computed(() => {
      const dayMenu = customMenu[selectedDay.value];
      return (
        dayMenu.Entrée.length + dayMenu.Plat.length + dayMenu.Dessert.length
      );
    });

    return {
      days,
      selectedDay,
      isLoading,
      customMenu,
      selectDay,
      addDish,
      importDishesFromApi,
      totalDishes,
      deleteDish,
    };
  },
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  margin: 40;
  padding: 0;
  background-color: #f8f9fa;
}

h1 {
  text-align: center;
  margin-top: 20px;
  color: #343a40;
}

h2,
h3,
p {
  display: flex;
  justify-content: center;
}

#api-btn {
  display: block;
  margin: 20px auto;
  padding: 10px 20px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  text-align: center;
}

#api-btn:hover {
  background-color: #218838;
}
</style>
