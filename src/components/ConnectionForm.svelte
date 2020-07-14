<script>
  import { slide, fade } from "svelte/transition";
  import { onMount, getContext } from "svelte";
  import { createSolaceClient, solaceContextKey } from "./solace-client";
  import { createMachine } from "@xstate/fsm";
  import { useMachine } from "./useMachine";

  let detailSectionOpen = true;

  function toggleDetailSection() {
    if (detailSectionOpen) {
      detailSectionOpen = false;
      return;
    }
    detailSectionOpen = true;
  }

  const connectMachine = createMachine({
    id: "connectMachine",
    initial: "disconnected",
    states: {
      disconnected: {
        on: {
          CONNECT_REQUEST: "connecting",
        },
      },
      connecting: {
        on: {
          UP_NOTICE: "connected",
          ERROR: "error",
        },
      },
      connected: {
        on: {
          DISCONNECTED: "disconnected",
        },
      },
      error: {
        on: {
          CONNECT_REQUEST: "connecting",
        },
      },
    },
  });

  const { state, send } = useMachine(connectMachine, {});

  const { getSolaceClient } = getContext(solaceContextKey);
  let solaceClient = getSolaceClient();

  let url = "ws://mr6cmufo3kilp.messaging.solace.cloud:80";
  let vpnName = "ajr";
  let userName = "solace-cloud-client";
  let password = "ig8pf95kah4appnglvkrgn2hot";

  async function handleConnect() {
    let _solaceClient = createSolaceClient({
      url,
      vpnName,
      userName,
      password,
    });
    _solaceClient.setOnUpNotice(() => {
      send("UP_NOTICE");
    });
    _solaceClient.setOnConnectFailedError(() => {
      send("ERROR");
    });
    _solaceClient.setOnDisconnected(() => {
      send("DISCONNECTED");
    });

    send("CONNECT_REQUEST");
    $solaceClient = await _solaceClient.connect();
    console.dir($solaceClient);
  }

  async function handleDisconnect() {
    if ($solaceClient) {
      $solaceClient.disconnect();
    }
  }
</script>

<div class="px-4 py-5 overflow-hidden bg-white rounded-lg shadow sm:p-6">
  <div class="flex items-center ">
    <h2 class="text-xl text-gray-900 sm:text-2xl">
      Establish connection to Solace
    </h2>
    {#if $state.value == 'connected'}
      <span class="h-6 ml-2">
        <svg
          class="h-full"
          fill="none"
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          viewBox="0 0 24 24"
          stroke="currentColor">
          <path
            class="text-green-500 stroke-current"
            d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656
            5.656l1.102-1.101m-.758-4.899a4 4 0 005.656 0l4-4a4 4 0
            00-5.656-5.656l-1.1 1.1" />
        </svg>
      </span>
    {:else}
      <span class="h-6 ml-2">
        <svg
          class="h-full text-red-500"
          fill="none"
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          viewBox="0 0 24 24"
          stroke="currentColor">
          <path d="M6 18L18 6M6 6l12 12" />
        </svg>
      </span>
    {/if}
  </div>

  <!-- connection details -->
  {#if detailSectionOpen}
    <div transition:slide>
      <div class="mt-2">
        <label
          for="brokerUrl"
          class="block text-sm font-medium leading-5 text-gray-700">
          Broker URL
        </label>
        <div class="relative mt-1 rounded-md shadow-sm">
          <input
            id="brokerUrl"
            class="block w-full form-input sm:text-sm sm:leading-5"
            placeholder=""
            bind:value={url} />
        </div>
      </div>
      <div class="mt-2">
        <label
          for="messageVpn"
          class="block text-sm font-medium leading-5 text-gray-700">
          Message VPN
        </label>
        <div class="relative mt-1 rounded-md shadow-sm">
          <input
            id="messageVpn"
            class="block w-full form-input sm:text-sm sm:leading-5"
            placeholder=""
            bind:value={vpnName} />
        </div>
      </div>
      <div class="mt-2">
        <label
          for="clientUsername"
          class="block text-sm font-medium leading-5 text-gray-700">
          Client Username
        </label>
        <div class="relative mt-1 rounded-md shadow-sm">
          <input
            id="clientUsername"
            class="block w-full form-input sm:text-sm sm:leading-5"
            placeholder=""
            bind:value={userName} />
        </div>
      </div>
      <div class="mt-2">
        <label
          for="clientPassword"
          class="block text-sm font-medium leading-5 text-gray-700">
          Client Password
        </label>
        <div class="relative mt-1 rounded-md shadow-sm">
          <input
            id="clientPassword"
            class="block w-full form-input sm:text-sm sm:leading-5"
            placeholder=""
            bind:value={password} />
        </div>
      </div>
    </div>
  {/if}

  <div class="flex mt-2">
    <div class="mr-2">
      <button
        on:click={handleConnect}
        type="button"
        class="inline-flex items-center px-4 py-2 text-base font-medium leading-6 text-white transition duration-150 ease-in-out bg-green-600 border border-transparent rounded-md hover:bg-green-500 focus:outline-none focus:border-green-700 focus:shadow-outline-green active:bg-green-700">
        Connect
      </button>
    </div>
    <div class="mr-2">
      <button
        on:click={handleDisconnect}
        type="button"
        class="inline-flex items-center px-4 py-2 text-base font-medium leading-6 text-white transition duration-150 ease-in-out bg-red-600 border border-transparent rounded-md hover:bg-red-500 focus:outline-none focus:border-red-700 focus:shadow-outline-red active:bg-red-700">
        Disconnect
      </button>
    </div>
    <div class="flex items-center justify-end flex-grow">
      <div>
        {#if detailSectionOpen}
          <button
            on:click={toggleDetailSection}
            class={`text-blue-500 transition duration-150 ease-in-out border border-transparent rounded-md outline-none focus:border-blue-700 focus:shadow-outline-blue focus:outline-none`}>
            hide
          </button>
        {:else}
          <button
            on:click={toggleDetailSection}
            class={`text-blue-500 transition duration-150 ease-in-out border border-transparent rounded-md outline-none focus:border-blue-700 focus:shadow-outline-blue focus:outline-none`}>
            show
          </button>
        {/if}
      </div>
    </div>
  </div>
</div>