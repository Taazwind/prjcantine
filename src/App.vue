<template>
  <div id="app">
    <h1>Cantine scolaire</h1>
    <button @click="toggleMenu">
      {{
        showCustomMenu
          ? "Afficher le menu chargé"
          : "Afficher le menu personnalisé"
      }}
    </button>
    <div v-if="showCustomMenu">
      <p>Nombre total de plats ajoutés : {{ totalCustomDishes }}</p>
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
      <div>
        <h2>Menu</h2>
        <div v-for="(dishes, category) in customMenu" :key="category">
          <h3>{{ category }}</h3>
          <div v-if="dishes.length > 0">
            <div v-for="dish in dishes" :key="dish.title">
              <h4>{{ dish.title }}</h4>
              <p>{{ dish.description }}</p>
              <p>Votes : {{ dish.votes }}</p>
              <button @click="dish.votes++">Voter</button>
            </div>
          </div>
          <p v-else>Aucun plat disponible</p>
        </div>
      </div>
    </div>
    <div v-else>
      <p>Nombre total de plats ajoutés : {{ totalApiDishes }}</p>
      <div>
        <h2>Menu API</h2>
        <div v-if="isLoading">
          <p>Chargement des plats en cours...</p>
        </div>
        <div v-else>
          <div v-for="(dishes, category) in apiMenu" :key="category">
            <h3>{{ category }}</h3>
            <div v-if="dishes.length > 0">
              <div v-for="dish in dishes" :key="dish.id">
                <h4>{{ dish.title }}</h4>
                <p>{{ dish.description }}</p>
                <p>Votes : {{ dish.votes }}</p>
                <button @click="dish.votes++">Voter</button>
              </div>
            </div>
            <p v-else>Aucun plat disponible</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, ref, reactive, computed } from "vue";

export default {
  setup() {
    const showCustomMenu = ref(true);
    const isLoading = ref(true);
    const newDish = reactive({
      title: "",
      description: "",
      category: "Entrée",
    });
    const customMenu = reactive({
      Entrée: [],
      Plat: [],
      Dessert: [],
    });
    const apiMenu = reactive({
      Entrée: [],
      Plat: [],
      Dessert: [],
    });

    const toggleMenu = () => {
      showCustomMenu.value = !showCustomMenu.value;
    };

    const addDish = () => {
      if (newDish.title && newDish.description) {
        customMenu[newDish.category].push({
          title: newDish.title,
          description: newDish.description,
          votes: 0,
        });
        newDish.title = "";
        newDish.description = "";
        newDish.category = "Entrée";
      }
    };

    const loadApiMenu = async () => {
      try {
        const response = await fetch(
          "https://tasty.p.rapidapi.com/recipes/list?from=0&size=10",
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
        data.results.forEach((recipe) => {
          const dish = {
            id: recipe.id,
            title: recipe.name,
            description: recipe.description || "Pas de description disponible",
            votes: 0,
          };

          if (isDessert(recipe)) {
            apiMenu.Dessert.push(dish);
          } else if (isStarter(recipe)) {
            apiMenu.Entrée.push(dish);
          } else {
            apiMenu.Plat.push(dish);
          }
        });
      } catch (error) {
        console.error("Erreur lors du chargement des plats :", error);
      } finally {
        isLoading.value = false;
      }
    };

    const isDessert = (recipe) => {
      const keywords = [
        "cake",
        "tart",
        "ice cream",
        "chocolate",
        "dessert",
        "muffins",
      ];
      return keywords.some((word) =>
        recipe.name.toLowerCase().includes(word)
      );
    };

    const isStarter = (recipe) => {
      const keywords = ["salad", "soup", "starter", "entrée"];
      return keywords.some((word) =>
        recipe.name.toLowerCase().includes(word)
      );
    };

    const totalCustomDishes = computed(
      () =>
        customMenu.Entrée.length +
        customMenu.Plat.length +
        customMenu.Dessert.length
    );

    const totalApiDishes = computed(
      () =>
        apiMenu.Entrée.length + apiMenu.Plat.length + apiMenu.Dessert.length
    );

    onMounted(loadApiMenu);

    return {
      showCustomMenu,
      isLoading,
      newDish,
      customMenu,
      apiMenu,
      toggleMenu,
      addDish,
      totalCustomDishes,
      totalApiDishes,
    };
  },
};
</script>

<style>
h1 {
  text-align: center;
}
form {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}
input,
select {
  margin-right: 10px;
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
</style>
