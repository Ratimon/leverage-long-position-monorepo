<script lang="ts">
  import WalletAccess from '$lib/blockchain/WalletAccess.svelte';
  import NavButton from '$lib/components/styled/navigation/NavButton.svelte';
  import Blockie from '$lib/components/generic/CanvasBlockie.svelte';
  import {wallet, flow, chain, fallback} from '$lib/blockchain/wallet';
  import {BigNumber} from '@ethersproject/bignumber';
  import {formatUnits} from '@ethersproject/units';
  import {onMount} from 'svelte';
  import {combine} from 'leverage-long-position-monorepo-common';

  let balanceInEth;
  let valueToSendInWEI;
  let isPositionActive;

  async function openPosition() {
    flow.execute(async (contracts) => {
      try {
        let tx = await wallet.contracts.LongETHPosition.openPosition({
          value: BigNumber.from(`${valueToSendInWEI}`).mul('1000000000000000000'),
        });
        await tx.wait();
      } catch (e) {
        console.log(e);
      }
    });
  }

  async function selectMaxvalue() {
    valueToSendInWEI = await fetchBalance();
    valueToSendInWEI = BigNumber.from(valueToSendInWEI).div('1000000000000000000').toNumber();
  }

  async function fetchBalance(): Promise<BigNumber> {
    const provider = wallet.provider || wallet.fallbackProvider;

    try {
      return await provider.getBalance(wallet.address);
    } catch (err) {
      console.log(err);
    }
  }

  async function fetchIsPositionActive(): Promise<boolean> {
    const contracts = wallet.contracts || fallback.contracts;
    if (!contracts) {
      return Promise.reject('no contract');
    }
    // return contracts.LongETHPosition.isCurrentPositionActive()
    //   .then((v) => {
    //     return fetch(v).then((r) => r.json());
    //     // return JSON.parse(v.substr('data:application/json,'.length));
    //   })
    //   .catch((e) => {
    //     console.log(e);
    //   });

    try {
      return await contracts.LongETHPosition.isCurrentPositionActive();
    } catch (err) {
      console.log(err);
    }
  }

  onMount(() => {
    console.log('mount openPosition', {
      combine: combine(wallet.address || '0x0000000000000000000000000000000000000000', 'hi').toString(),
    });
  });

  chain.subscribe(async ($chain) => {
    if (!balanceInEth && $chain.state === 'Ready') {
      const balanceInWei = await fetchBalance();
      balanceInEth = formatUnits(balanceInWei, 18);
    }
  });

  fallback.subscribe(async ($fallback) => {
    if (!balanceInEth && $fallback.state === 'Ready') {
      const balanceInWei = await fetchBalance();
      balanceInEth = formatUnits(balanceInWei, 18);
      // balanceInEth = BigNumber.from(balanceInWei).div('1000000000000000000');
    }
  });

  chain.subscribe(async ($chain) => {
    if (!isPositionActive && $chain.state === 'Ready') {
      isPositionActive = await fetchIsPositionActive();
    }
  });

  fallback.subscribe(async ($fallback) => {
    if (!isPositionActive && $fallback.state === 'Ready') {
      isPositionActive = await fetchIsPositionActive();
    }
  });

  function formatError(error): string {
    try {
      return JSON.stringify(error, null, '  ');
    } catch (e) {
      return e.message || e;
    }
  }
</script>

<section class="py-2 px-4 text-center">
  <div class="max-w-md mx-auto pt-1 mt-5 space-y-3 md:mt-8 md:space-y-5">
    <!-- <h2 class="text-3xl tracking-tight font-extrabold text-gray-900 dark:text-gray-100 sm:text-4xl">Leverage</h2>

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
          class="big secondary"
          disabled={!valueToSendInWEI || valueToSendInWEI === 0 || typeof valueToSendInWEI != 'number'}
          on:click={openPosition}
        >
          Leverage 1.3
        </NavButton>
      </div>
    </form> -->

    <WalletAccess>
      {#if $chain.state === 'Ready' || $fallback.state === 'Ready'}
        <!-- TODO pregenerate so it can always be viewable without a node-->

        <h2 class="text-3xl tracking-tight font-extrabold text-gray-900 dark:text-gray-100 sm:text-4xl">Leverage</h2>

        {#if isPositionActive == false}
          {#await isPositionActive}
            <!-- wait for the first time -->
            <h2 class="text-3xl tracking-tight font-extrabold text-gray-900 dark:text-gray-100 sm:text-4xl">Loading</h2>
            <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
              Current Balance: {balanceInEth} ETH
            </p>
            -->
          {:then}
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
                  class="big secondary"
                  disabled={!valueToSendInWEI || valueToSendInWEI === 0 || typeof valueToSendInWEI != 'number'}
                  on:click={openPosition}
                >
                  Leverage 1.3
                </NavButton>
              </div>
            </form>
          {:catch error}
            <p style="color: red">{formatError(error)}</p>
          {/await}
        {:else}
          <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
            The Position has not been opened yet. Must wait for being closed first !!!
          </p>
        {/if}

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
      {:else}
        <p class="m-6 text-gray-500 dark:text-gray-400 text-xl">Please connect to interact</p>

        <NavButton
          class="w-24 mx-auto"
          label="Connect"
          disabled={$wallet.unlocking || $chain.connecting}
          on:click={() => flow.connect()}
        >
          Connect
        </NavButton>
      {/if}
    </WalletAccess>
  </div>
</section>
