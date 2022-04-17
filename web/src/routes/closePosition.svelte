<script lang="ts">
  import WalletAccess from '$lib/blockchain/WalletAccess.svelte';
  import NavButton from '$lib/components/styled/navigation/NavButton.svelte';
  import Blockie from '$lib/components/generic/CanvasBlockie.svelte';
  import {wallet, flow, chain, fallback} from '$lib/blockchain/wallet';
  import {BigNumber} from '@ethersproject/bignumber';
  import {onMount} from 'svelte';
  import {combine} from 'leverage-long-position-monorepo-common';

  const name = 'Leverage';
  let message = '';
  let balanceInEth;
  let valueToSendInWEI;

  async function setMessage() {
    await flow.execute((contracts) => contracts.GreetingsRegistry.setMessage(message));
  }

  async function selectMaxvalue() {
    // valueToSend = await fetchBalance();

    valueToSendInWEI = await fetchBalance();
    //
    valueToSendInWEI = BigNumber.from(valueToSendInWEI).div('1000000000000000000').toNumber();
  }

  async function openPosition() {
    flow.execute(async (contracts) => {
      try {
        console.log('wallet', wallet);
        console.log('wallet', contracts);

        let tx = await wallet.contracts.LongETHPosition.openPosition({
          value: BigNumber.from(`${valueToSendInWEI}`).mul('1000000000000000000'),
        });
        await tx.wait();
      } catch (e) {
        console.log(e);
      }
    });

    // const balanceInWei = await fetchBalance();
    // balanceInEth = BigNumber.from(balanceInWei).div('1000000000000000000');
  }

  async function fetchBalance(): Promise<BigNumber> {
    const provider = wallet.provider || wallet.fallbackProvider;

    // const contracts = wallet.contracts || fallback.contracts;
    // if (!contracts) {
    //   return Promise.reject('no contract');
    // }

    try {
      return await provider.getBalance(wallet.address);
    } catch (err) {
      console.log(err);
    }
  }

  //   function convertToETH(underlyingAmount) {
  //     return underlyingAmount;
  //   }

  onMount(() => {
    console.log('mount demo', {
      combine: combine(wallet.address || '0x0000000000000000000000000000000000000000', 'hi').toString(),
    });
  });

  //   wallet.subscribe(async ($wallet) => {
  //     if (!balance && $chain.state === 'Ready') {
  //       balance = $wallet.balance;
  //       // lastAddress = $wallet.address;
  //       //     Sentry.setUser({address: $wallet.address});
  //     }
  //   });

  chain.subscribe(async ($chain) => {
    if (!balanceInEth && $chain.state === 'Ready') {
      const balanceInWei = await fetchBalance();
      balanceInEth = BigNumber.from(balanceInWei).div('1000000000000000000');
    }
  });

  fallback.subscribe(async ($fallback) => {
    if (!balanceInEth && $fallback.state === 'Ready') {
      const balanceInWei = await fetchBalance();
      balanceInEth = BigNumber.from(balanceInWei).div('1000000000000000000');
    }
  });
</script>

<section class="py-2 px-4 text-center">
  <div class="max-w-auto md:max-w-lg mx-auto">
    <h2 class="text-6xl font-black mb-2 font-heading text-black dark:text-white">
      {name}
    </h2>

    <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
      Current Balance: {balanceInEth} ETH
    </p>

    <form class="w-full max-w-sm">
      <div class="flex items-center border-b border-pink-600 py-2">
        <input
          class="appearance-none bg-transparent border-none w-full text-gray-700 dark:text-gray-300 mr-3 py-1 px-2
                      leading-tight focus:outline-none"
          type="number"
          placeholder="0.0 ETH"
          aria-label="value to send"
          bind:value={valueToSendInWEI}
        />
        <button
          class="flex-shrink-0 bg-pink-600 hover:bg-pink-700 border-pink-600 hover:border-pink-700 text-sm border-4
                      text-white py-1 px-2 rounded disabled:bg-gray-400 disabled:border-gray-400 disabled:cursor-not-allowed"
          type="button"
          on:click={selectMaxvalue}
        >
          MAX
        </button>
      </div>
      <div class="flex items-center">
        <NavButton
          label="Leverage 1.3"
          disabled={!valueToSendInWEI || valueToSendInWEI === 0 || typeof valueToSendInWEI != 'number'}
          on:click={openPosition}
        >
          Leverage 1.3
        </NavButton>
      </div>
    </form>

    <WalletAccess>
      <!-- <form class="w-full max-w-sm">
          <div class="flex items-center border-b border-pink-600 py-2">
            <input
              class="appearance-none bg-transparent border-none w-full text-gray-700 dark:text-gray-300 mr-3 py-1 px-2
                      leading-tight focus:outline-none"
              type="text"
              placeholder="0.0"
              aria-label="Your Message"
              bind:value={message}
            />
            <button
              disabled={!valueToSendInWEI || valueToSendInWEI === 0 || typeof valueToSendInWEI != 'number'}
              on:click={openPosition}
              class="flex-shrink-0 bg-pink-600 hover:bg-pink-700 border-pink-600 hover:border-pink-700 text-sm border-4
                      text-white py-1 px-2 rounded disabled:bg-gray-400 disabled:border-gray-400 disabled:cursor-not-allowed"
              type="button"
            >
              Say It!
            </button>
          </div>
        </form> -->

      {#if $wallet.state === 'Ready'}
        <form class="mt-5 w-full max-w-sm">
          <div class="flex items-center">
            <NavButton
              label="Disconnect"
              disabled={$wallet.unlocking || $chain.connecting}
              on:click={() => wallet.disconnect()}
            >
              Disconnect
            </NavButton>
          </div>
        </form>
      {/if}
    </WalletAccess>

    <!-- <div class="pt-3 pb-4 dark:bg-black bg-white">
        <h1 class="dark:text-gray-500 text-gray-500 m-4 font-semibold">Use it:</h1>
        <code id="leverage-long-position-monorepo-command" on:click={select} class="mb-5 text-pink-600 font-black"
          >npx degit wighawag/leverage-long-position-monorepo your-app-folder</code
        >
        <p class="mt-6 text-gray-500">
          Find out more on
          <a
            class="underline"
            href="https://github.com/wighawag/leverage-long-position-monorepo#readme"
            target="_blank"
            rel="noopener">github</a
          >
        </p>
      </div> -->
  </div>
</section>
