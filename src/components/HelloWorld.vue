<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <button v-if="done.length === 0" @click="getAnswer">Click</button>
    <h5>progress: {{ done }}</h5>

    <h1>Transactions</h1>

    <h5>{{ answer }}</h5>

    <h1>Balances</h1>

    <h5>{{ balances }}</h5>
  </div>
</template>

<script>
const axios = require("axios");

export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data: () => ({
    answer: {},
    balances: {},
    done: ""
  }),
  methods: {
    readProducts() {
      console.log("Hello");
    },
    async getAnswer() {
      this.done = "working!";

      const address = "0xEcA19B1a87442b0c25801B809bf567A6ca87B1da";
      var link =
        "https://api-ropsten.etherscan.io/api?module=account&action=tokentx&address=";
      var query =
        "&startblock=0&endblock=999999999&sort=asc&apikey=YourApiKeyToken";
      var queue = [address];
      var visited = new Set();
      // var balances = [
      //   { address: "0xEcA19B1a87442b0c25801B809bf567A6ca87B1da", balance: 0 }
      // ];
      var transactions = [];
      while (queue.length > 0) {
        const { data } = await axios.get(link + queue[0] + query);
        // var transList = []
        if (data.status == 1) {
          const filtered = data.result.filter(a => a.tokenSymbol === "BKTC");
          const formatted = filtered.map(a => {
            return { hash: a.hash, to: a.to, from: a.from, amount: a.value };
          });
          formatted.forEach(item => {
            if (transactions.filter(p => p.hash == item.hash).length === 0) {
              transactions.push(item);
              queue.push(item.to);
            }
          });
          // console.log(transferTotal);
          console.log(queue.length, "left to check");
          visited.add(queue[0]);
          queue.shift();
          // console.log(transactions);
          // console.log(filtered);
          this.answer = Array.from(transactions);
        }
      }
      let pos = Array.from(visited);

      let balances = pos.map(a => {
        return { address: a, balance: 0 };
      });
      transactions.forEach(item => {
        balances[pos.indexOf(item.to)].balance += item.amount;
        balances[pos.indexOf(item.from)].balance -= item.amount;
        if (balances[pos.indexOf(item.from)] <= 0) {
          balances[pos.indexOf(item.from)].balance = 0;
        }
      });
      console.log("DONE!");
      this.done = "done!";
      this.balances = balances;

      //Suppose we have all the transactions
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
