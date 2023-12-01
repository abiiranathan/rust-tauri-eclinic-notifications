<script lang="ts">
	import { invoke } from '@tauri-apps/api/tauri';
	import {
		isPermissionGranted,
		requestPermission,
		sendNotification
	} from '@tauri-apps/api/notification';

	import { onDestroy, onMount } from 'svelte';

	type MessageNotification = {
		channel: string;
		message: string;
		time: string;
	};

	let eventSource: EventSource | undefined;
	let connectionOpen = false;
	let messages: MessageNotification[] = [];

	$: {
		if (eventSource) {
			connectionOpen = eventSource.readyState == eventSource.OPEN;
		}
	}

	onMount(async () => {
		const serverUrl = await invoke<string>('get_server_url');

		eventSource = new EventSource(serverUrl);
		eventSource.addEventListener('message', handleMessage, true);

		eventSource.addEventListener(
			'open',
			() => {
				connectionOpen = true;
				console.log('connectionOpen');
			},
			true
		);

		eventSource.addEventListener(
			'error',
			() => {
				connectionOpen = false;
				console.log('connectionError');
			},
			true
		);

		document.addEventListener('contextmenu', disableContextMenu);
	});

	function disableContextMenu(event: MouseEvent) {
		event.preventDefault();
		event.stopImmediatePropagation();
	}

	onDestroy(async () => {
		if (eventSource) {
			eventSource.close();
		}
		document.removeEventListener('contextmenu', disableContextMenu);
	});

	function handleMessage(event: MessageEvent) {
		try {
			const payload: MessageNotification = JSON.parse(event.data);
			if (payload.channel == 'KeepAlive' || payload.message == '') {
				return;
			}

			messages = [payload, ...messages];
			sendUserNotification(payload);
		} catch (error) {
			console.error(error);
		}
	}

	async function sendUserNotification(message: MessageNotification) {
		let permissionGranted = await isPermissionGranted();
		if (!permissionGranted) {
			const permission = await requestPermission();
			permissionGranted = permission === 'granted';
		}

		if (permissionGranted) {
			sendNotification({ title: message.channel, body: message.message });
		}
	}
</script>

<div class="container">
	<div class="header">
		<h2>Notifications</h2>
		<p style="padding: 1rem; color: green;">
			{messages.length}
			{messages.length == 1 ? 'message' : 'messages'}
		</p>
		{connectionOpen ? 'Online' : 'Offline'}
		<button on:click={() => (messages = [])}>CLEAR MESSAGES</button>
		<div class="status" class:open={connectionOpen} class:closed={!connectionOpen}></div>
	</div>
	<div class="content">
		{#each messages as message}
			<div class="message">
				<div class="message_header">
					<div class="title">{message.channel}</div>
					<div class="timestamp">{new Date(message.time).toLocaleTimeString('en-GB')}</div>
				</div>
				<div class="text">{message.message}</div>
			</div>
		{/each}
	</div>
</div>

<style>
	*,
	*::before {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	:global(body) {
		min-height: 100vh;
		height: 100vh;
		margin: 0;
		display: flex;
		flex-direction: column;
		box-sizing: border-box;
		font-size: 1rem;
		overflow: hidden;
	}
	h2 {
		font-weight: normal;
	}

	.container {
		display: grid;
		align-items: stretch;
		grid-template-rows: auto 1fr; /* Updated to have two rows */
		justify-content: stretch;
		gap: 0.5rem;
		flex-grow: 1; /* Allow container to expand vertically */
		padding: 1rem;
		height: 100%;
	}

	.header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		max-width: 90%;
		width: 100%;
		margin: auto;
	}
	.status {
		width: 20px;
		height: 20px;
		border-radius: 50%;
		animation: blink 1s ease infinite 500ms;
	}

	@keyframes blink {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}

	.status.open {
		background-color: green;
	}

	.status.closed {
		background-color: red;
	}

	.content {
		width: 100%;
		margin: 1rem auto;
		border: 1px solid #ccc;
		max-height: 100%;
		overflow-y: auto;
		flex-grow: 1; /* Allow content to expand to fill remaining height */
		border-top-right-radius: 1rem;
		border-top-left-radius: 1rem;
	}

	.message {
		padding: 10px;
		border-bottom: 1px solid #bdb8b8;
		background-color: #f8f8f8;
	}

	.message_header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 0.2rem 0.5rem;
	}

	.message_header .title {
		font-weight: 800;
		font-size: 1.2rem;
		color: rgb(47, 40, 40);
	}

	.message .text {
		font-size: 1rem;
		padding-top: 0.4rem;
		padding: 0.2rem 0.5rem;
	}

	button {
		padding: 0.25rem 1.25rem;
		background-color: #dbd8d8;
		color: #232323;
		border: 1px solid #ccc;
		border-radius: 1.25rem;
		cursor: pointer;
	}
	button:hover {
		background-color: lightblue;
		color: teal;
	}
</style>
