<script setup>
import { ref } from 'vue'
import { Engine } from 'json-rules-engine'
import Exchange from './components/Exchange.vue'

const engine = new Engine();
let points = ref(0)
const items = [{ id: 1, name: 'Water', price: 50 },
{ id: 2, name: 'Pepsi', price: 150 },
{ id: 3, name: 'Tea', price: 350 },
{ id: 4, name: 'Coffee', price: 550 }]
const shopCart = ref([])
let summ = ref()
let exchangeMessage = ref()

function add(id) {
  if (!shopCart.value.some(item => item.id === id)) {
    items.forEach((item) => {
      if (item.id === id) {
        item.count = 1
        item.total = item.count * item.price
        shopCart.value.push(item)
      }
    })
  } else {
    shopCart.value.forEach((item) => {
      if (item.id === id) {
        item.count += 1;
        item.total = item.count * item.price
      }
    })
  }
  summ.value = shopCart.value.map(item => item.total).reduce((prev, curr) => prev + curr, 0)
}

engine.addRule({
  conditions: {
    all: [
      {
        fact: 'summ',
        operator: 'lessThan',
        value: 100,
      }
    ]
  },
  event: {
    type: 'exchange',
    params: {
      points:0
    }
  }
});

engine.addRule({
  conditions: {
    all: [
      {
        fact: 'summ',
        operator: 'greaterThanInclusive',
        value: 100
      },
      {
        fact: 'summ',
        operator: 'lessThan',
        value: 500,
      },
    ]
  },
  event: {
    type: 'exchange',
    params: {
      points:100
    }
  }
});

engine.addRule({
  conditions: {
    all: [
      {
        fact: 'summ',
        operator: 'greaterThanInclusive',
        value: 500
      },
      {
        fact: 'summ',
        operator: 'lessThan',
        value: 1000,
      },
    ]
  },
  event: {
    type: 'exchange',
    params: {
      points:500
    }
  }
});

engine.addRule({
  conditions: {
    all: [
      {
        fact: 'summ',
        operator: 'greaterThanInclusive',
        value: 1000
      }
    ]
  },
  event: {
    type: 'exchange',
    params: {
      points:1000
    }
  }
});

function buy() {
  const options = {
    summ: +summ.value
  };

  engine
    .run(options)
    .then(({ events }) => {
      events.map(event => {
        exchangeMessage.value =`You get ${event.params.points} points`
        points.value += event.params.points
      })
      shopCart.value.splice(0)
      summ.value = null
      setTimeout(() => exchangeMessage.value = null, 1000)
    });
}
</script>

<template>
  <h1> {{ points }} Points </h1>
  <div class="exchange-wrapper">
    <Exchange />
  </div>
  <h4 style="position: absolute; top: 0%; right: 5%;">{{ exchangeMessage }}</h4>
  <div class="shop">
    <div class="shop-items-all">
      <h2>Assortment</h2>
      <div class='item' v-for="item in items">
        <h3 style="width: 33%">{{ item.name }} </h3>
        <h3 style="width: 33%">{{ item.price }} $</h3>
        <button style="width: 70px; width: 33%;" @click="() => add(item.id)">add</button>
      </div>

    </div>
    <div class="basket">
      <h2>Shopping cart</h2>
      <div class='item' v-for="item in shopCart">
        <h3 style="width: 25%">{{ item.name }} </h3>
        <h3 style="width: 25%">{{ item.price }} $</h3>
        <h3 style="width: 25%">{{ item.count }}</h3>
        <h3 style="width: 25%">{{ item.total }} $</h3>
        <!-- <button style="width: 70px; width: 33%;" @click="add(item.id)">add</button> -->
      </div>
      <button v-if="summ" style="background-color: green;" @click="buy">{{ summ }} $</button>
    </div>
  </div>
</template>

<style scoped>
.item {
  margin: 0 auto;
  width: 90%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border: 1px solid violet;
  border-radius: 20px;
  padding-right: 15px;
  margin-bottom: 10px;
}

.exchange-wrapper {
  display: flex;
  justify-content: center;
}

.shop {
  display: flex;
  justify-content: center;
}

.shop-items-all,
.basket {
  width: 450px;
  height: 450px;
  border-radius: 20px;
  /* background-color: gray; */
  margin: 10px;
  border: 1px solid salmon;

}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
