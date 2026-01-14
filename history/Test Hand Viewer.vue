<template>
  <div class="app-container">
    <div class="header">
      <h1>Bridge Hands Viewer</h1>
      <p><strong>Subfolder:</strong> {{ currentHand.Subfolder }}</p>
      <p><strong>Deal Number:</strong> {{ currentHand.DealNumber }}</p>
    </div>

    <div class="hand-display">
      <div class="hand-column">
        <h3>North</h3>
        <p v-html="formatHand(currentHand.NorthHand)"></p>
      </div>
      <div class="horizontal-hands">
        <div class="hand-row">
          <h3>West</h3>
          <p v-html="formatHand(currentHand.WestHand)"></p>
        </div>
        <div class="hand-row">
          <h3>East</h3>
          <p v-html="formatHand(currentHand.EastHand)"></p>
        </div>
      </div>
      <div class="hand-column">
        <h3>South</h3>
        <p v-html="formatHand(currentHand.SouthHand)"></p>
      </div>
    </div>

    <div class="navigation">
      <button @click="prevHand" :disabled="currentIndex === 0">Previous</button>
      <button @click="nextHand" :disabled="currentIndex === hands.length - 1">Next</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      hands: [
        { Subfolder: "100Deals", Filename: "deal001.html", DealNumber: "1", NorthHand: "S:J86 H:A98 D:8653 C:AKQ", EastHand: "S:A9 H:QT54 D:QJT2 C:JT3", SouthHand: "S:Q53 H:762 D:AK4 C:8742", WestHand: "S:KT742 H:KJ3 D:97 C:965" },
        { Subfolder: "100Deals", Filename: "deal002.html", DealNumber: "2", NorthHand: "S:74 H:KQJ3 D:AK75 C:863", EastHand: "S:J986 H:A5 D:Q32 C:QJT4", SouthHand: "S:AKQ3 H:T982 D:J4 C:K72", WestHand: "S:T52 H:764 D:T986 C:A95" },
        { Subfolder: "100Deals", Filename: "deal003.html", DealNumber: "3", NorthHand: "S:AQ5 H:74 D:Q962 C:AJT8", EastHand: "S:KT H:QJT982 D:83 C:652", SouthHand: "S:872 H:K6 D:AKJ4 C:KQ74", WestHand: "S:J9643 H:A53 D:T75 C:93" },
        { Subfolder: "100Deals", Filename: "deal004.html", DealNumber: "4", NorthHand: "S:KQJ H:A862 D:Q63 C:T94", EastHand: "S:72 H:QT43 D:AJ92 C:Q73", SouthHand: "S:T98653 H:K7 D:84 C:AKJ", WestHand: "S:A4 H:J95 D:KT75 C:8652" },
        // Add additional hands from your sample as needed
      ],
      currentIndex: 0, // Index of the current hand being displayed
    };
  },
  computed: {
    currentHand() {
      return this.hands[this.currentIndex] || {};
    },
  },
  methods: {
    formatHand(hand) {
      if (!hand) return "";
      return hand
        .replace(/S:/g, "♠ ")
        .replace(/H:/g, '<span class="red">♥</span> ')
        .replace(/D:/g, '<span class="red">♦</span> ')
        .replace(/C:/g, "♣ ");
    },
    nextHand() {
      if (this.currentIndex < this.hands.length - 1) {
        this.currentIndex++;
      }
    },
    prevHand() {
      if (this.currentIndex > 0) {
        this.currentIndex--;
      }
    },
  },
};
</script>

<style scoped>
/* General layout and responsiveness */
.app-container {
  padding: 20px;
  font-family: Arial, sans-serif;
  text-align: center;
  max-width: 600px;
  margin: auto;
}

.header {
  margin-bottom: 20px;
}

.hand-display {
  display: grid;
  grid-template-columns: 1fr;
  gap: 10px;
}

.hand-column {
  text-align: center;
}

.horizontal-hands {
  display: flex;
  justify-content: space-between;
}

.hand-row {
  flex: 1;
  margin: 0 5px;
  text-align: center;
}

.red {
  color: red;
}

.navigation {
  margin-top: 20px;
}

button {
  padding: 10px 20px;
  margin: 5px;
  border: none;
  background-color: #007bff;
  color: white;
  font-size: 16px;
  cursor: pointer;
  border-radius: 5px;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

button:hover:enabled {
  background-color: #0056b3;
}

/* Responsive layout for smaller screens */
@media (max-width: 600px) {
  .horizontal-hands {
    flex-direction: column;
  }

  .hand-row {
    margin-bottom: 10px;
  }
}
</style>