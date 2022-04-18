<script lang="ts">
  import WalletAccess from '$lib/blockchain/WalletAccess.svelte';
  import NavButton from '$lib/components/styled/navigation/NavButton.svelte';
  import Blockie from '$lib/components/generic/CanvasBlockie.svelte';
  import {wallet, flow, chain, fallback} from '$lib/blockchain/wallet';
  import {BigNumber} from '@ethersproject/bignumber';
  import {formatUnits} from '@ethersproject/units';

  import {onMount} from 'svelte';
  import {combine} from 'leverage-long-position-monorepo-common';

  let depositAmount;
  let borrowAmount;
  let totalETH;
  let UniSwapOutput;
  let currentETHPrice;
  let pnl;
  let isPositionActive;

  async function closePosition() {
    flow.execute(async (contracts) => {
      try {
        let tx = await wallet.contracts.LongETHPosition.closePosition();
        await tx.wait();
      } catch (e) {
        console.log(e);
      }
    });
  }

  async function fetchIsPositionActive(): Promise<boolean> {
    const contracts = wallet.contracts || fallback.contracts;
    if (!contracts) {
      return Promise.reject('no contract');
    }

    try {
      return await contracts.LongETHPosition.isCurrentPositionActive();
    } catch (err) {
      console.log(err);
    }
  }

  async function fetchDepositAmount(): Promise<BigNumber> {
    const contracts = wallet.contracts || fallback.contracts;
    if (!contracts) {
      return Promise.reject('no contract');
    }
    try {
      return await contracts.LongETHPosition.getCurrentDepositAmount();
    } catch (err) {
      console.log(err);
    }
  }

  async function fetchBorrowAmount(): Promise<BigNumber> {
    const contracts = wallet.contracts || fallback.contracts;
    if (!contracts) {
      return Promise.reject('no contract');
    }

    try {
      return await contracts.LongETHPosition.getCurrentBorrowAmount();
    } catch (err) {
      console.log(err);
    }
  }

  async function fetchUniSwapOutput(): Promise<BigNumber> {
    const contracts = wallet.contracts || fallback.contracts;
    if (!contracts) {
      return Promise.reject('no contract');
    }

    try {
      return await contracts.LongETHPosition.getExpectedUniSwapOutput();
    } catch (err) {
      console.log(err);
    }
  }

  async function fetchTotalExposure(): Promise<BigNumber> {
    const contracts = wallet.contracts || fallback.contracts;
    if (!contracts) {
      return Promise.reject('no contract');
    }

    try {
      return await contracts.LongETHPosition.getTotalExposure();
    } catch (err) {
      console.log(err);
    }
  }

  async function fetchCurrentEthPrice(): Promise<BigNumber> {
    const contracts = wallet.contracts || fallback.contracts;
    if (!contracts) {
      return Promise.reject('no contract');
    }

    try {
      return await contracts.LongETHPosition.getCurrentETHPrice();
    } catch (err) {
      console.log(err);
    }
  }

  async function fetchPNL(): Promise<BigNumber> {
    const contracts = wallet.contracts || fallback.contracts;
    if (!contracts) {
      return Promise.reject('no contract');
    }

    try {
      return await contracts.LongETHPosition.getExpectedProfitInUsd();
    } catch (err) {
      console.log(err);
    }
  }

  onMount(() => {
    console.log('mount closePosition', {
      combine: combine(wallet.address || '0x0000000000000000000000000000000000000000', 'hi').toString(),
    });
  });

  chain.subscribe(async ($chain) => {
    if (!depositAmount && $chain.state === 'Ready') {
      const depositInWei = await fetchDepositAmount();
      //   depositAmount = BigNumber.from(depositInWei).div('1000000000000000000');
      depositAmount = formatUnits(depositInWei, 18);

      //   depositAmount = await fetchDepositAmount();
    }
  });

  fallback.subscribe(async ($fallback) => {
    if (!depositAmount && $fallback.state === 'Ready') {
      const depositInWei = await fetchDepositAmount();
      //   depositAmount = BigNumber.from(depositInWei).div('1000000000000000000');
      depositAmount = formatUnits(depositInWei, 18);
      //   depositAmount = await fetchDepositAmount();
    }
  });

  chain.subscribe(async ($chain) => {
    if (!borrowAmount && $chain.state === 'Ready') {
      const borrowInWei = await fetchBorrowAmount();
      borrowAmount = BigNumber.from(borrowInWei).div('1000000000000000000');
      //   borrowAmount = formatUnits(borrowInWei, 18);
    }
  });

  fallback.subscribe(async ($fallback) => {
    if (!borrowAmount && $fallback.state === 'Ready') {
      const borrowInWei = await fetchBorrowAmount();
      borrowAmount = BigNumber.from(borrowInWei).div('1000000000000000000');
      //   borrowAmount = formatUnits(borrowInWei, 18);
    }
  });

  chain.subscribe(async ($chain) => {
    if (!UniSwapOutput && $chain.state === 'Ready') {
      const uniSwapOutputInWei = await fetchUniSwapOutput();
      UniSwapOutput = BigNumber.from(uniSwapOutputInWei).div('1000000000000000000');
      //   UniSwapOutput = await fetchUniSwapOutput();
    }
  });

  fallback.subscribe(async ($fallback) => {
    if (!UniSwapOutput && $fallback.state === 'Ready') {
      const uniSwapOutputInWei = await fetchUniSwapOutput();
      UniSwapOutput = BigNumber.from(uniSwapOutputInWei).div('1000000000000000000');
      //   UniSwapOutput = await fetchUniSwapOutput();
    }
  });

  chain.subscribe(async ($chain) => {
    if (!totalETH && $chain.state === 'Ready') {
      const totalETHInWei = await fetchTotalExposure();
      totalETH = BigNumber.from(totalETHInWei).div('1000000000000000000');
      //   totalETH = await fetchTotalExposure();
    }
  });

  fallback.subscribe(async ($fallback) => {
    if (!totalETH && $fallback.state === 'Ready') {
      const totalETHInWei = await fetchTotalExposure();
      totalETH = BigNumber.from(totalETHInWei).div('1000000000000000000');
      //   totalETH = await fetchTotalExposure();
    }
  });

  chain.subscribe(async ($chain) => {
    if (!currentETHPrice && $chain.state === 'Ready') {
      currentETHPrice = await fetchCurrentEthPrice();
    }
  });

  fallback.subscribe(async ($fallback) => {
    if (!currentETHPrice && $fallback.state === 'Ready') {
      currentETHPrice = await fetchCurrentEthPrice();
    }
  });

  chain.subscribe(async ($chain) => {
    if (!pnl && $chain.state === 'Ready') {
      //   const pnlInWei = await fetchPNL();
      //   pnl = BigNumber.from(pnlInWei).div('1000000000000000000');
      pnl = await fetchPNL();
    }
  });

  fallback.subscribe(async ($fallback) => {
    if (!pnl && $fallback.state === 'Ready') {
      //   const pnlInWei = await fetchPNL();
      //   pnl = BigNumber.from(pnlInWei).div('1000000000000000000');
      pnl = await fetchPNL();
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
    <!-- <h2 class="text-3xl tracking-tight font-extrabold text-gray-900 dark:text-gray-100 sm:text-4xl">My Position</h2> -->

    <!-- <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
      Deposit Amount: {depositAmount} ETH
    </p>

    <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
      Borrow Amount: {borrowAmount} ETH
    </p>

    <h2 class="text-3xl tracking-tight font-extrabold text-gray-900 dark:text-gray-100 sm:text-4xl">
      Swap {borrowAmount} to {UniSwapOutput}
    </h2>

    <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
      Total ETH: {totalETH} ETH
    </p> -->

    <!-- <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">Current ETH Price : {currentETHPrice} ETH</p>

    <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">PNL : {pnl} USD</p> -->

    <!-- <div class="max-w-md mx-auto pt-1 mt-5 space-y-3 md:mt-8 md:space-y-5">
      <div class="space-y-5 sm:flex sm:justify-center sm:space-y-0 sm:space-x-3">
        <NavButton label="Close Position" blank={true} class="big secondary" on:click={closePosition}>
          Close Position
        </NavButton>
      </div>
    </div> -->

    <!-- <div class="flex items-center">
        <NavButton
          label="Leverage 1.3"
          class="big secondary"
          disabled={!valueToSendInWEI || valueToSendInWEI === 0 || typeof valueToSendInWEI != 'number'}
          on:click={openPosition}
        >
          Leverage 1.3
        </NavButton>
      </div> -->

    <WalletAccess>
      {#if $chain.state === 'Ready' || $fallback.state === 'Ready'}
        <!-- TODO pregenerate  so it can always be viewable without a node-->
        <h2 class="text-3xl tracking-tight font-extrabold text-gray-900 dark:text-gray-100 sm:text-4xl">
          The Position
        </h2>

        {#if isPositionActive == true}
          {#await isPositionActive}
            <!-- <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
              Current Balance: {balanceInEth} ETH
            </p> -->
          {:then}
            <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
              Deposit Amount: {depositAmount} ETH
            </p>

            <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
              Borrow Amount: {borrowAmount} DAI
            </p>

            <h2 class="text-3xl tracking-tight font-extrabold text-gray-900 dark:text-gray-100 sm:text-4xl">
              Swap {borrowAmount} DAI to {UniSwapOutput} ETH
            </h2>

            <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
              Total ETH: {totalETH} ETH
            </p>

            <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">Current ETH Price : {currentETHPrice} USD</p>

            <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">PNL : {pnl} USD</p>

            <div class="max-w-md mx-auto pt-1 mt-5 space-y-3 md:mt-8 md:space-y-5">
              <div class="space-y-5 sm:flex sm:justify-center sm:space-y-0 sm:space-x-3">
                <NavButton label="Close Position" blank={true} class="big secondary" on:click={closePosition}>
                  Close Position
                </NavButton>
              </div>
            </div>
          {:catch error}
            <p style="color: red">{formatError(error)}</p>
          {/await}
        {:else}
          <p class="m-6 text-gray-800 dark:text-gray-300 text-xl">
            The Position has not been opened yet. Must wait for being opened first !!!
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
