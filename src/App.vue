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
        <h2>Menu Personnalisé</h2>
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
      <p>Nombre total de plats chargés : {{ totalApiDishes }}</p>
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
export default {
  data() {
    return {
      showCustomMenu: true,
      isLoading: true,
      newDish: {
        title: "",
        description: "",
        category: "Entrée",
      },
      customMenu: {
        Entrée: [],
        Plat: [],
        Dessert: [],
      },
      apiMenu: {
        Entrée: [],
        Plat: [],
        Dessert: [],
      },
    };
  },
  computed: {
    totalCustomDishes() {
      return (
        this.customMenu.Entrée.length +
        this.customMenu.Plat.length +
        this.customMenu.Dessert.length
      );
    },
    totalApiDishes() {
      return (
        this.apiMenu.Entrée.length +
        this.apiMenu.Plat.length +
        this.apiMenu.Dessert.length
      );
    },
  },
  methods: {
    toggleMenu() {
      this.showCustomMenu = !this.showCustomMenu;
    },
    addDish() {
      if (this.newDish.title && this.newDish.description) {
        this.customMenu[this.newDish.category].push({
          title: this.newDish.title,
          description: this.newDish.description,
          votes: 0,
        });
        this.newDish.title = "";
        this.newDish.description = "";
        this.newDish.category = "Entrée";
      }
    },
    async loadApiMenu() {
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

          if (this.isDessert(recipe)) {
            this.apiMenu.Dessert.push(dish);
          } else if (this.isStarter(recipe)) {
            this.apiMenu.Entrée.push(dish);
          } else {
            this.apiMenu.Plat.push(dish);
          }
        });
      } catch (error) {
        console.error("Erreur lors du chargement des plats :", error);
      } finally {
        this.isLoading = false;
      }
    },
    isDessert(recipe) {
      const keywords = ["cake", "tart", "ice cream", "chocolate", "dessert", "muffins"];
      return keywords.some((word) => recipe.name.toLowerCase().includes(word));
    },
    isStarter(recipe) {
      const keywords = ["salad", "soup", "starter", "entrée"];
      return keywords.some((word) => recipe.name.toLowerCase().includes(word));
    },
  },
  async mounted() {
    await this.loadApiMenu();
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
