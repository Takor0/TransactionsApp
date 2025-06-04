<template>
  <div class="candidate-table-container">
    <h3>Real-Time Transactions</h3>
    <div class="table-wrapper">
      <RecycleScroller
        :items="displayedTransactions"
        :item-size="rowHeight"
        key-field="n"
        class="virtual-tbody"
        :buffer="buffer"
      >
        <template #before>
          <tr>
            <th>Time</th>
            <th>Price</th>
            <th>Volume</th>
            <th>Value</th>
          </tr>
        </template>
        <template #default="{ item }">
          <td>{{ item.t }}</td>
          <td>{{ item.p }}</td>
          <td>{{ item.v }}</td>
          <td>{{ item.u }}</td>
        </template>
      </RecycleScroller>
      <div v-if="isLoading" class="loading-overlay">
        <div class="loader" aria-hidden="true"></div>
        <span class="loading-text">Loading initial data...</span>
      </div>
    </div>
    <div class="stats">
      Total Transactions: {{ displayedTransactions.length }}
    </div>
  </div>
</template>
<script>
import { RecycleScroller } from 'vue-virtual-scroller';
let transactionCounter = 0;

function generateTransaction() {
  const now = new Date();
  const time = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}:${now.getSeconds().toString().padStart(2, '0')}.${now.getMilliseconds().toString().padStart(3, '0')}`;
  const price = (Math.random() * 100 + 100).toFixed(2);
  const volume = Math.floor(Math.random() * 500) + 1;
  const value = (price * volume).toFixed(2);
  return {
    n: transactionCounter++,
    t: time,
    p: price,
    v: volume,
    u: value
  };
}


export default {
  props: {},
  components: { RecycleScroller },
  data() {
    return {
      transactions: [],
      isLoading: true,
      updateInterval: null,
      initialBatchSize: 10000,
      updateIntervalMs: 200,
      buffer: 20,
      rowHeight: 40
    };
  },
  computed: {
    displayedTransactions() {
      return this.transactions;
    }
  },
  methods: {
    loadInitialData() {
      console.log(`Generating ${this.initialBatchSize} initial transactions...`);
      const initialBatch = [];
      for (let i = 0; i < this.initialBatchSize; i++) {
        initialBatch.push(generateTransaction());
      }
      this.transactions = initialBatch.reverse();
      this.isLoading = false;
      console.log("Initial data loaded.");
      this.startStreamingUpdates();
    },
    startStreamingUpdates() {
      if (this.updateInterval) {
        clearInterval(this.updateInterval);
      }
      console.log(`Starting streaming updates (1 transaction every ${this.updateIntervalMs}ms)...`);
      this.updateInterval = setInterval(() => {
        const newTransaction = generateTransaction();
        this.transactions.unshift(newTransaction);
      }, this.updateIntervalMs);
    },
    stopStreamingUpdates() {
      if (this.updateInterval) {
        clearInterval(this.updateInterval);
        this.updateInterval = null;
        console.log("Stopped streaming updates.");
      }
    }
  },
  mounted() {
    this.loadInitialData();
  },
  beforeUnmount() {
    this.stopStreamingUpdates();
  }
};
</script>
<style>
.vue-recycle-scroller__slot {
  display: table-header-group;
  width: 100%;
}
.vue-recycle-scroller__item-wrapper {
  display: table-row-group;
}
.vue-recycle-scroller__item-view {
  display: table-row;
}
.virtual-tbody {
  display: table;
  width: 100%;
}
.candidate-table-container {
  max-width: 800px;
  margin: auto;
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #f2f2f2;
}

td {
  font-family: monospace;
}

tr:hover {
  background-color: #f1f1f1;
}

.table-wrapper {
  height: 400px;
  overflow-y: auto;
  position: relative;
}

.stats {
  margin-top: 10px;
  font-size: 14px;
  color: #555;
}

.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(255, 255, 255, 0.8);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  color: #333;
  z-index: 1;
}

.loader {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin-bottom: 10px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
