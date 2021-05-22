<template>
  <div class="hello">
    <input v-model="address" placeholder="Address"/> <button @click="load">load containers</button>

    <div v-if="address">
      <div>
        <h3>containers of {{ address }}:</h3>

        <ul>
          <li v-for="token in tokens" :key="token">
            container N{{ token }} <button @click="read(token)">read what inside</button>
            <div v-if="balances[token]">
              <h3>tokens of container {{ token }}</h3>

              <h4>erc20 tokens:</h4>
              <ul>
                <li v-for="erc20name in Object.keys(balances[token])" :key="token+erc20name">
                  {{ erc20name }}: {{ balances[token][erc20name] }}
                </li>
              </ul>
            </div>
          </li>
        </ul>

        <div v-if="tokens.length === 0">
          no tokens at this address
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {
  defineComponent, reactive, ref, watch,
} from 'vue';
import Web3 from 'web3';
import abi from '../assets/main_abi';

const web3 = new Web3('https://data-seed-prebsc-1-s1.binance.org:8545');

const registry = '0xb5df2c8c4449caf1df5d5cfbe4c7ddf6d73588b2';
const locker20 = '0x5ba89bC29CFb41E60D57640BE848010893715BE5';
const locker721 = '0x5ba89bC29CFb41E60D57640BE848010893715BE5';

const erc20: Record<string, string|number>[] = [{
  name: 'busd',
  id: '1353855610468061688490308507012873682781345878766',
}];

const erc721 = {
  busdId: '1353855610468061688490308507012873682781345878766',
};

const createContract = (from: string) => new web3.eth.Contract(abi as any, registry, {
  from, // default from address
  gasPrice: '20000000000', // default gas price in wei, 20 gwei in this case
});

const MAX_TOKEN_ID = 999;

export default defineComponent({
  name: 'PandoraReader',

  setup() {
    const address = ref('0x6be7D3398c7f701deFc42EC4930533e5aDF883BA');
    const tokens = reactive([] as string[]);
    const balances = reactive({} as Record<string, Record<string, string>>);

    let pandora = createContract(address.value);

    watch(address, (value) => {
      pandora = createContract(value);
    });

    const load = async () => {
      let index = 0;

      while (true) {
        // eslint-disable-next-line no-await-in-loop
        const tokenId = await pandora.methods
          // eslint-disable-next-line no-plusplus
          .tokenOfOwnerByIndex(address.value, index++).call() as string;

        if (Number(tokenId) > MAX_TOKEN_ID) {
          break;
        }

        tokens.push(tokenId);
      }
    };

    const read20 = async (tokenId: string) => {
      balances[tokenId] = {};

      for (let i = 0; i < erc20.length; i += 1) {
        const token = erc20[i];

        // eslint-disable-next-line no-await-in-loop
        const res = await pandora
          .methods.childBalance(tokenId, locker20, token.id).call() as string;

        balances[tokenId][token.name] = res;
      }

      return balances;
    };

    const read = async (tokenId: string) => {
      const erc20balances = await read20(tokenId);
      // const erc721 = await read721(tokenId);

      console.log(erc20balances);
    };

    return {
      load,
      address,
      tokens,
      read,
      balances,
      erc20,
    };
  },
});
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
