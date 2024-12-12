<template>
  <div id="app">
    <h1>Cantine scolaire</h1>

    <div class="day-buttons">
      <button v-for="day in days" :key="day" @click="selectDay(day)">
        {{ day }}
      </button>
    </div>

    <h2>Menu du {{ selectedDay }}</h2>

    <div v-if="showCustomMenu">
      <form @submit.prevent="addDish">
        <input v-model="newDish.title" placeholder="Nom du plat" />
        <input v-model="newDish.description" placeholder="Description" />
        <select v-model="newDish.category">
          <option>Entrée</option>
          <option>Plat</option>
          <option>Dessert</option>
        </select>
        <button type="submit">Ajouter</button>
      </form>

      <button id="coubeh" @click="importDishesFromApi" :disabled="isLoading">
        {{ isLoading ? "Chargement..." : "Importer des plats depuis l'API" }}
      </button>

      <div>
        <p>Nombre total de plats ajoutés : {{ totalDishes }}</p>
        <h2>Menu</h2>
        <div
          v-for="(dishes, category) in customMenu[selectedDay]"
          :key="category"
        >
          <h3>{{ category }}</h3>
          <div class="dish-container" v-if="dishes.length > 0">
            <div class="dish-box" v-for="dish in dishes" :key="dish.title">
              <h4>{{ dish.title }}</h4>
              <p>{{ dish.description }}</p>
              <p>Votes : {{ dish.votes }}</p>
              <button @click="dish.votes++">Voter</button>
              <button @click="deleteDish(category, dish)" class="btn del-btn">
                Supprimer
              </button>
            </div>
          </div>
          <p v-else>Aucun plat disponible</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive, computed } from "vue";

export default {
  setup() {
    const days = ["Lundi", "Mardi", "Mercredi", "Jeudi", "Vendredi"];
    const selectedDay = ref("Lundi");
    const showCustomMenu = ref(true);
    const isLoading = ref(false);

    const newDish = reactive({
      title: "",
      description: "",
      category: "Entrée",
    });

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

    const addDish = () => {
      if (newDish.title && newDish.description) {
        customMenu[selectedDay.value][newDish.category].push({
          title: newDish.title,
          description: newDish.description,
          votes: 0,
        });
        newDish.title = "";
        newDish.description = "";
        newDish.category = "Entrée";
      }
    };

    const displayedDishes = reactive({
      Entrée: [],
      Plat: [],
      Dessert: [],
    });

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

        // Sélectionner une entrée, un plat et un dessert aléatoire
        const starter = findUniqueDish(shuffledRecipes, "Entrée");
        const mainDish = findUniqueDish(shuffledRecipes, "Plat");
        const dessert = findUniqueDish(shuffledRecipes, "Dessert");

        if (starter && mainDish && dessert) {
          // Ajouter ces plats au menu du jour
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

          // Ajouter ces plats à la liste des plats affichés
          displayedDishes["Entrée"].push(starter.name);
          displayedDishes["Plat"].push(mainDish.name);
          displayedDishes["Dessert"].push(dessert.name);
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
        if (
          dishFinder(recipe) &&
          !displayedDishes[category].includes(recipe.name)
        ) {
          return recipe;
        }
      }
      return null;
    };

    // Fonction de mélange aléatoire (Fisher-Yates)
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
      showCustomMenu,
      isLoading,
      newDish,
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

form {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

h2,
h3,
p {
  display: flex;
  justify-content: center;
}

input,
select {
  margin-right: 10px;
  padding: 5px;
  border: 1px solid #ced4da;
  border-radius: 3px;
}

button {
  background-color: #28a745;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 3px;
}

button:hover {
  background-color: #218838;
}

#coubeh {
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

#coubeh:hover {
  background-color: #218838;
}

.dish-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
  margin: 20px 0;
}

.dish-box {
  background-color: white;
  border: 1px solid #dee2e6;
  border-radius: 5px;
  padding: 15px;
  width: 250px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.dish-box h4 {
  margin: 0;
  color: #495057;
}

.dish-box p {
  margin: 5px 0;
  color: #6c757d;
}

.dish-box button {
  margin-top: 10px;
  background-color: #007bff;
}

.dish-box button:hover {
  background-color: #0056b3;
}

.day-buttons {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  gap: 10px;
}

.day-buttons button {
  padding: 10px 15px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.day-buttons button:hover {
  background-color: #0056b3;
}

.btn.del-btn {
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 3px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.btn.del-btn:hover {
  background-color: #c82333;
}
</style>
