<script>
	import { invoke } from '@tauri-apps/api/tauri';
	import { goto } from '$app/navigation';

	import Button from '../lib/component/Button.svelte';
	import Toggle from '../lib/component/Toggle.svelte';
	import { userContext, UserContext, UserRank } from '../stores.ts';
	import { addNotification, Notification } from '$lib/notif/NotifHandler';

	let uid;

	async function sign_in() {
		// TODO: backend implementation
		await invoke('login', {
			uid: uid,
		})
			.then(async (response) => {
				let userData = new UserContext();
				userData.serialize(response);
				userContext.update((_) => userData);
				localStorage.setItem(
					'userContextData',
					JSON.stringify(response)
				);
				goto('/launcher');
			})
			.catch((err) => {
				let errorMessage = null;
				// Possibly find the error message in the error json
				for (const error in err) {
					errorMessage = err[error].message;
				}
				// If there is no message an unknown error occurred
				if (errorMessage == null) {
					errorMessage =
						'An unknown error occurred, please create an issue on GitHub';
				}
				addNotification(
					new Notification(
						'Login Error',
						errorMessage,
						`error`,
						30e3,
						true,
						[
							{
								name: 'Copy',
								callback: 'copy-text',
								metadata: errorMessage,
							},
						]
					)
				);
				console.log(errorMessage);
				console.log(err);
			});
	}

	function on_remember_update(new_state) {
		// TODO: backend implementation
		invoke('set_remember_state', {
			state: new_state,
		});
	}
</script>

<main class="flex flex-col h-full justify-center items-center">
	<img
		width="45px"
		src="https://moonclient.xyz/logo.png"
		class="mb-1 mr-1"
		alt="branding"
	/>
	<div class="flex flex-row items-center mb-6">
		<p class="text-lg text-gray-200">
			<b style="font-weight: 800">Moon</b> Client
		</p>
	</div>

	<div
		class="bg-slate-700/[0.25] border border-slate-50/[0.15] w-80 max-w-md rounded-lg px-4 py-4"
		style="backdrop-filter: blur(100px)"
	>
		<h2 class="font-bold text-center text-2xl">Sign in</h2>
		<h2 class="text-center text-gray-300 text-xs">to the launcher</h2>

		<div class="space-y-2">
			<label class="flex flex-col mt-4 text-sm font-semibold">
				<span class="mb-1">UID</span>
				<input
					class="bg-slate-800/[0.6] border border-slate-50/[0.15] border rounded-lg mt-0.5 p-2.5 transition-colors outline-0 focus:border-blue-500 focus:ring-blue-500 block"
					required
					bind:value={uid}
				/>
			</label>

			<Toggle
				text="Remember me"
				default_state="true"
				on:click={on_remember_update}
			/>

			<Button text="Sign in" on:click={sign_in} />
		</div>
	</div>
</main>
