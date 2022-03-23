<script lang="ts">
	import { onMount } from 'svelte';

  let baudRate: number = 9600;
  let messages: string[] = [];
  let isPortOpen: boolean = false;

	const getPorts = () => {
    navigator.serial
      .getPorts()
      .then((ports) => {
        console.log('Ports:', ports);
      })
      .catch((err) => {
        console.error(err);
      });
	};

	const openPort = async () => {
		try {
			const port = await navigator.serial.requestPort({
				// filters: [{ usbVendorId: 0x10c4 }]
			});
			await port.open({ baudRate });
      isPortOpen = true;
			while (port.readable) {
				const reader = port.readable.getReader();
				try {
          let currentMessage = '';
					while (true) {
						const { value, done } = await reader.read();
						if (done) {
							// |reader| has been canceled.
							break;
						}
						// Do something with |value|...
            if (value) {
              if (value[0] == '\r'.charCodeAt(0)) continue;
              if (value[0] == '\n'.charCodeAt(0)) {
                messages = [...messages, currentMessage];
                currentMessage = '';
                continue;
              }
              currentMessage += String.fromCharCode(value[0]);
            }
					}
				} catch (error) {
					// Handle |error|...
          console.error(error);
          
				} finally {
					reader.releaseLock();
          isPortOpen = false;
				}
			}
		} catch (e) {
			console.log(e);
		}
	};

	onMount(getPorts);
</script>

<h1>Svelte Serial reader</h1>
<p>Status: {isPortOpen ? 'connected' : 'disconnected'}</p>
<label for="baudRate">Baudrate</label>
<input name="baudRate" type="text" bind:value={baudRate}>
<button on:click={openPort}>Open port</button>

<h2>Incoming messages</h2>
{#if messages.length > 0}
<ul>
  {#each messages as message}
  <li>{message}</li>
  {/each}
</ul>
{/if}