<script setup>
import { ref } from 'vue'
import { Engine } from 'json-rules-engine'

let sodaCounter = ref(0)
let bottleCounter = ref(0)
let exchangeMessage = ref()
const engine = new Engine();

engine.addRule({
    conditions: {
        all: [
            {
                fact: 'emptyBottlesCount',
                operator: 'greaterThanInclusive',
                value: 2
            }
        ]
    },
    event: {
        type: 'exchange',
        params: {
            message: 'Exchanged!'
        }
    }
});

engine.addRule({
    conditions: {
        all: [{
            fact: 'emptyBottlesCount',
            operator: 'lessThan',
            value: 2,
        }]
    },
    event: {
        type: 'notEnoughBottles',
        params: {
            message: "Need more empty bottles for an exchange!",
        }
    }
});


function exchange() {
    const options = {
        emptyBottlesCount: bottleCounter.value,
    };

    engine
        .run(options)
        .then(({ events }) => {
            events.map(event => exchangeMessage.value = event.params.message)
            sodaCounter.value += Math.floor(bottleCounter.value / 2)
            bottleCounter.value = 0
            setTimeout(() => exchangeMessage.value = null, 1000)
        });
}

</script>

<template>
    <div>
        <h4 style="position: absolute; top: 0%; right: 5%;">{{ exchangeMessage }}</h4>
        <h2>You have {{ sodaCounter ? sodaCounter : 0 }} soda</h2>
        <div class="exchanger">
            <div class="exchange-bottle">
                <h3>Bottle {{ bottleCounter }}</h3>
                <button style="width: 70px;" @click="bottleCounter++">add</button>
                <button style="width: 70px; background-color: brown;"
                    @click="() => { if (bottleCounter > 0) bottleCounter-- }">del</button>
            </div>

            <button style="width: 150px; background-color: green; height: 42.4px;" @click="exchange">Exchange</button>
        </div>
    </div>
</template>

<style>
.exchanger {
    width: 500px;
    display: flex;
    justify-content: space-around;
    flex-wrap: nowrap;
    align-items: center;
}

.exchange-bottle {
    display: flex;
    width: 250px;
    justify-content: space-between;
    align-items: center;
}
</style>