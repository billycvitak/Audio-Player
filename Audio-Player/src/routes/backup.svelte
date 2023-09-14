<script lang="ts">
	import { onDestroy, onMount } from 'svelte';
	import { slide } from 'svelte/transition';
	import { quintInOut } from 'svelte/easing';
	import { filterValueStore, visualizerStore } from '$lib/store';
	import type { Writable } from 'svelte/store';

	import PlaybackControls from '$lib/Player/PlaybackControls.svelte';
	import PlayerDisplay from '$lib/Player/PlayerDisplay.svelte';
	import Playlist from '$lib/Player/Playlist.svelte';
	import Settings from '$lib/Tuning/Settings.svelte';
	import VisualizerBar from '$lib/Visualizer/VisualizerBar.svelte';
	import VisualizerWaveform from '$lib/Visualizer/VisualizerWaveform.svelte';
	import ShortcutModal from '$lib/Tuning/ShortcutModal.svelte';
	import Notification from '$lib/Tuning/Notification.svelte';

	let audioFileInput: HTMLInputElement;
	let audio: HTMLAudioElement;
	let audioContext: AudioContext | null = null;
	let audioSource: MediaElementAudioSourceNode | null = null;
	let panNode: StereoPannerNode | null = null;
	let analyser: AnalyserNode | null = null;
	let dataArray: Uint8Array | null = null;
	let animationFrameId: number | null = null;

	let playlist: Array<{ name: string; url: string }> = [];
	let currentTrackIndex = -1;
	let currentTrackName = '';
	let isPlaying = false;
	let isMuted = false;
	let loopMode = 1;
	let loopModeLabel = '';
	let volume = 1;
	let balance: number;
	let leftChannelVolume = 1;
	let rightChannelVolume = 1;
	let duration = 0;
	let currentTime = 0;
	let seekPosition = 0;
	let isModalOpen = false;
	let isLeftDrawerOpen = false;
	let isRightDrawerOpen = false;
	let isBottomDrawerOpen = false;
	let filterValue: string;
	let visualizerValue: string | Writable<string>;
	let message = '';
	let notificationsEnabled = true;
	let defaultSong = {
		name: 'Winamp Demo Audio',
		url: '/song.mp3',
		duration: '04:20',
		format: 'mp3',
		kbps: '320 kbps'
	};
	const numBars = 45;
	const maxBarHeight = 100;
	const maxRecentlyPlayed = 3;
	const recentlyPlayed: number[] = [];

	const unsubscribe = filterValueStore.subscribe((value) => {
		filterValue = value;
		visualizerStore.subscribe((value) => {
			visualizerValue = value;
		});
	});

	$: visualizerValue;

	// PLAYLIST //

	function loadPlaylist(event: Event) {
		const files = (event.target as HTMLInputElement).files;
		if (files && files.length > 0) {
			const newTracks: Array<{
				name: string;
				url: string;
				duration: string;
				kbps: string;
				format: string;
			}> = [];

			for (let i = 0; i < files.length; i++) {
				const file = files[i];
				const objectURL = URL.createObjectURL(file);

				const audioElement = new Audio();
				audioElement.src = objectURL;

				audioElement.addEventListener('loadedmetadata', () => {
					const trackInfo = {
						name: file.name,
						url: objectURL,
						duration: formatTime(audioElement.duration),
						kbps: calculateKbps(file.size, audioElement.duration),
						format: getFileFormat(file.name)
					};

					newTracks.push(trackInfo);

					if (newTracks.length === files.length) {
						playlist = playlist.concat(newTracks);

						if (playlist.length > 0) {
							if (currentTrackIndex === -1) {
								loadTrack(0);
							}
						}
					}
				});
			}
		}
	}

	function shufflePlaylist() {
		if (playlist.length > 1) {
			let randomIndex;

			do {
				randomIndex = Math.floor(Math.random() * playlist.length);
			} while (recentlyPlayed.includes(randomIndex));

			if (recentlyPlayed.length >= maxRecentlyPlayed) {
				recentlyPlayed.shift();
			}

			recentlyPlayed.push(randomIndex);
			loadTrack(randomIndex);
		}
	}

	function loadTrack(trackIndex: number) {
		if (trackIndex >= 0 && trackIndex < playlist.length) {
			currentTrackIndex = trackIndex;

			const track = playlist[currentTrackIndex];

			audio.src = track.url;
			audio.load();

			currentTrackName = track.name;

			if (audioContext && audioContext.state === 'suspended') {
				audioContext.resume().then(() => {
					audio.play();
				});
			} else {
				audio.play();
			}

			duration = audio.duration;

			isPlaying = true;

			renderEqualizerVisualizer();
		}
	}

	function clearPlaylist() {
		currentTrackIndex = -1;
		playlist = [];
		audio.src = '';
		audio.currentTime = 0;
		duration = 0;
		stopVisualizer();
		if (isPlaying) {
			audio.pause();
			isPlaying = false;
		}
		audioFileInput.value = '';
		audioFileInput.removeEventListener('change', loadPlaylist);
		audioFileInput.addEventListener('change', loadPlaylist);
	}

	function removeSong(index: number) {
		if (currentTrackIndex === index) {
			audio.pause();
			audio.src = '';
			audio.currentTime = 0;
			currentTrackName = 'No Track Playing';
			duration = 0;
			isPlaying = false;
			currentTrackIndex = -1;
		} else if (currentTrackIndex > index) {
			currentTrackIndex--;
		}

		playlist = playlist.filter((_, i) => i !== index);
	}

	// CONTROLS //

	function togglePlayback() {
		if (isPlaying) {
			audio.pause();
		} else {
			audio.play();
		}
		isPlaying = !isPlaying;
	}

	function moveToNextTrack() {
		if (currentTrackIndex >= playlist.length - 1) {
			loadTrack(0);
		} else {
			loadTrack(currentTrackIndex + 1);
		}
	}

	function increaseVolume() {
		if (volume < 1) {
			volume += 0.1;
			audio.volume = volume;
		}
	}

	function decreaseVolume() {
		if (volume > 0) {
			volume -= 0.1;
			audio.volume = volume;
		}
	}

	function changeVolume(event: { target: { value: string } }) {
		volume = parseFloat(event.target.value);
		audio.volume = volume;
	}

	function toggleMute() {
		if (isMuted) {
			audio.volume = 1;
			volume = 1;
		} else {
			audio.volume = 0;
			volume = 0;
		}
		isMuted = !isMuted;
	}

	function seek() {
		// TOREDO, bug; doesnt work on my machine
		let newCurrentTime = (seekPosition / 100) * duration;
		audio.currentTime = newCurrentTime;

		if (isPlaying) {
			audio.pause();
			audio.play();
		} else {
			audio.play().then(() => {
				isPlaying = true;
				audio.pause();
			});
		}
	}

	function toggleLoopMode() {
		loopMode = (loopMode + 1) % 3;
		updateLoopModeLabel();
	}

	function updateLoopModeLabel() {
		if (loopMode === 0) {
			loopModeLabel = 'Loop: Off';
		} else if (loopMode === 1) {
			loopModeLabel = 'Loop: Playlist';
		} else if (loopMode === 2) {
			loopModeLabel = 'Loop: Current Track';
		}
	}

	function changeVolumeBalance(event: { target: { value: string } }) {
		balance = parseFloat(event.target.value);

		if (panNode) {
			if (balance <= 0.5) {
				leftChannelVolume = 1;
				rightChannelVolume = 2 * balance;
			} else {
				leftChannelVolume = 2 * (1 - balance);
				rightChannelVolume = 1;
			}

			updateAudioBalance();
		}
	}

	function updateAudioBalance() {
		if (panNode) {
			panNode.pan.value = balance * 2 - 1;
		}
	}

	// VISUALIZER BAR //

	function updateBarHeights() {
		analyser?.getByteFrequencyData(dataArray!);

		for (let i = 0; i < numBars; i++) {
			const bar = document.getElementById(`bar-${i}`) as HTMLDivElement | null;
			if (bar) {
				const barHeight = (dataArray![i] / 420) * maxBarHeight;
				bar.style.height = `${barHeight}%`;
			}
		}
	}

	// VISUALIZER WAVEFORM //

	function updateWaveform() {
		analyser?.getByteTimeDomainData(dataArray!);

		const waveformCanvas = document.getElementById('waveform') as HTMLCanvasElement | null;
		if (waveformCanvas) {
			const ctx = waveformCanvas.getContext('2d');
			if (ctx) {
				ctx.clearRect(0, 0, waveformCanvas.width, waveformCanvas.height);
				ctx.beginPath();
				const sliceWidth = waveformCanvas.width / dataArray!.length;
				let x = 0;

				for (let i = 0; i < dataArray!.length; i++) {
					const v = dataArray![i] / 128.0;
					const y = (v * waveformCanvas.height) / 2;

					if (i === 0) {
						ctx.moveTo(x, y);
					} else {
						ctx.lineTo(x, y);
					}

					x += sliceWidth;
				}

				ctx.lineTo(waveformCanvas.width, waveformCanvas.height / 2);
				ctx.strokeStyle = 'rgba(0, 0, 0, 0.7)';
				ctx.lineWidth = 2;
				ctx.stroke();
			}
		}
	}

	// ACCESSIBILITY //

	function toggleNotifications() {
		notificationsEnabled = !notificationsEnabled;
	}

	const showNotification = debounce((msg: string) => {
		if (notificationsEnabled) message = msg;
		setTimeout(() => {
			message = '';
		}, 1000);
	}, 100);

	const handleKeyDown = (event: { key: string; shiftKey: any }) => {
		if (event.key === ' ') {
			togglePlayback();
			if (isPlaying) showNotification('Play Music (Hotkey: SPACE)');
			else showNotification('Pause Music (Hotkey: SPACE)');
		} else if (event.shiftKey) {
			toggleBottomDrawer();
			if (isBottomDrawerOpen) showNotification('Visualizer On (Hotkey: SHIFT)');
			else showNotification('Visualizer Off (Hotkey: SHIFT)');
		} else if (event.key === 'M' || event.key === 'm') {
			toggleMute();
			if (isMuted) showNotification('Volume Muted (Hotkey: M)');
			else showNotification('Volume Unmuted (Hotkey: M)');
		} else if (event.key === 'ArrowUp') {
			increaseVolume();
			showNotification('Volume Up (Hotkey: ARROW_UP)');
		} else if (event.key === 'ArrowDown') {
			decreaseVolume();
			showNotification('Volume Down (Hotkey: ARROW_DOWN)');
		} else if (event.key === 'S' || event.key === 's') {
			shufflePlaylist();
			showNotification('Shuffle Next (Hotkey: S)');
		} else if (event.key === 'ArrowRight') {
			moveToNextTrack();
			showNotification('Next Track (Hotkey: ARROW_RIGHT)');
		} else if (event.key === 'ArrowLeft') {
			loadTrack(currentTrackIndex - 1);
			showNotification('Previous Track (Hotkey: ARROW_LEFT)');
		} else if (event.key === 'P' || event.key === 'p') {
			toggleRightDrawer();
			if (isRightDrawerOpen) showNotification('Open Playlist (Hotkey: P)');
			else showNotification('Close Playlist (Hotkey: P)');
		} else if (event.key === 'T' || event.key === 't') {
			toggleLeftDrawer();
			if (isLeftDrawerOpen) showNotification('Open Tuning (Hotkey: T)');
			else showNotification('Close Tuning (Hotkey: T)');
		} else if (event.key === 'H' || event.key === 'h') {
			isModalOpen = !isModalOpen;
		} else if (event.key === 'C' || event.key === 'c') {
			clearPlaylist();
			showNotification('Playlist Cleared (Hotkey: C)');
		} else if (event.key === 'L' || event.key === 'l') {
			toggleLoopMode();
			showNotification(loopModeLabel + ' (Hotkey: L)');
		}
	};

	// HELPERS //

	function initializeAudio() {
		if (!audioContext) {
			audioContext = new (window.AudioContext || (window as any).webkitAudioContext)();
			if (!audioContext) {
				console.error('Web Audio API is not supported in this browser.');
				return;
			}

			analyser = audioContext.createAnalyser();
			if (!analyser) {
				console.error('Analyser node creation failed.');
				return;
			}

			analyser.fftSize = 256;
			dataArray = new Uint8Array(analyser.frequencyBinCount);

			audioSource = audioContext.createMediaElementSource(audio);
			panNode = audioContext.createStereoPanner();

			audioSource.connect(analyser);
			analyser.connect(audioContext.destination);

			audioSource.connect(panNode);
			panNode.connect(audioContext.destination);
		}
	}

	function renderEqualizerVisualizer() {
		updateBarHeights();

		animationFrameId = requestAnimationFrame(renderEqualizerVisualizer);
	}

	function stopVisualizer() {
		if (animationFrameId) {
			cancelAnimationFrame(animationFrameId);
		}
	}

	function formatTime(seconds: number) {
		const minutes = Math.floor(seconds / 60);
		seconds = Math.floor(seconds % 60);
		return `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
	}

	function calculateKbps(fileSizeInBytes: number, durationInSeconds: number) {
		const bitsPerByte = 8;
		const bitsPerKb = 1024 * bitsPerByte;

		const fileSizeInBits = fileSizeInBytes * bitsPerByte;
		const kbps = (fileSizeInBits / durationInSeconds / 1000).toFixed(0);

		return `${kbps} kbps`;
	}

	function getFileFormat(filename: string) {
		const parts = filename.split('.');
		if (parts.length > 1) {
			return parts[parts.length - 1];
		}
		return 'N/A';
	}

	function debounce<T extends (...args: any[]) => void>(func: T, wait: number) {
		let timeout: NodeJS.Timeout;
		return (...args: Parameters<T>) => {
			clearTimeout(timeout);
			timeout = setTimeout(() => {
				func(...args);
			}, wait);
		};
	}

	function toggleModal() {
		isModalOpen = !isModalOpen;
	}

	function toggleLeftDrawer() {
		isLeftDrawerOpen = !isLeftDrawerOpen;
	}

	function toggleRightDrawer() {
		isRightDrawerOpen = !isRightDrawerOpen;
	}

	function toggleBottomDrawer() {
		isBottomDrawerOpen = !isBottomDrawerOpen;
	}

	onMount(() => {
		audio = document.getElementById('audio') as HTMLAudioElement;
		audioFileInput = document.getElementById('thefile') as HTMLInputElement;
		audioFileInput.addEventListener('change', loadPlaylist);
		audio.addEventListener('loadedmetadata', () => {
			duration = audio.duration;
		});

		audio.addEventListener('timeupdate', () => {
			currentTime = audio.currentTime;
			seekPosition = (audio.currentTime / duration) * 100;
		});

		audio.addEventListener('ended', () => {
			if (loopMode === 0) {
				isPlaying = false;
			} else if (loopMode === 1) {
				if (currentTrackIndex < playlist.length - 1) {
					loadTrack(currentTrackIndex + 1);
				} else {
					loadTrack(0);
				}
			} else if (loopMode === 2) {
				audio.currentTime = 0;
				audio.play();
			}
		});

		document.body.addEventListener('keydown', handleKeyDown);

		initializeAudio();

		updateLoopModeLabel();

		playlist = [defaultSong];

		loadTrack(0);
	});

	onDestroy(() => {
		unsubscribe();
		stopVisualizer();
		if (audioContext) {
			audioContext.close().catch(console.error);
		}
	});
</script>

<!-- Hidden -->
<audio id="audio" class="hidden" controls />
<input
	type="file"
	accept="audio/mpeg, audio/aac, audio/ogg, audio/wav, audio/webm, audio/flac, audio/x-m4a, audio/x-ms-wma, audio/aiff, audio/wave, audio/wav; codecs=1, audio/x-wav, audio/wave; codecs=1, audio/vnd.wav, audio/wave; codecs=0, audio/wav; codecs=0"
	id="thefile"
	multiple
	class="hidden"
/>

<div
	class="{filterValue} transition-all duration-500 flex w-full bg-black justify-center items-center h-screen bg-cover bg-center select-none font-mono subpixel-antialiased"
	style="background-image:url()"
>
	<!-- Main Container with Fixed Center -->
	<div class="flex">
		<!-- Left Drawer -->
		{#if isLeftDrawerOpen}
			<div
				class="h-96 w-96 text-white/50 rounded-lg transform-gpu pr-1"
				transition:slide={{
					delay: 250,
					duration: 300,
					easing: quintInOut,
					axis: 'x'
				}}
			>
				<Settings
					{toggleModal}
					{notificationsEnabled}
					{toggleNotifications}
					{toggleLoopMode}
					{loopModeLabel}
				/>
				<!-- Left Drawer Content -->
			</div>
		{:else}
			<div class="hidden" />
		{/if}

		<!-- Main Music Player Container -->
		<div class="h-96 w-96 rounded-lg">
			<PlayerDisplay
				{seek}
				{seekPosition}
				{playlist}
				{currentTime}
				{currentTrackName}
				{formatTime}
				{duration}
				{volume}
				{changeVolume}
				{changeVolumeBalance}
				{balance}
			/>
			<PlaybackControls
				{isPlaying}
				{moveToNextTrack}
				{togglePlayback}
				{loadTrack}
				{currentTrackIndex}
				{shufflePlaylist}
				{toggleRightDrawer}
				{toggleLeftDrawer}
				{toggleBottomDrawer}
			/>
			<!-- Bottom Drawer -->
			{#if isBottomDrawerOpen}
				<div
					class="w-96 rounded-lg transform-gpu text-white absolute"
					transition:slide={{
						delay: 250,
						duration: 300,
						easing: quintInOut,
						axis: 'y'
					}}
				>
					{#if visualizerValue === 'Bar'}
						<VisualizerBar {numBars} {isPlaying} />
					{:else if visualizerValue === 'Waveform'}
						<VisualizerWaveform />
					{/if}
				</div>
			{:else}
				<div class="hidden" />
			{/if}
		</div>

		<!-- Right Drawer -->
		{#if isRightDrawerOpen}
			<div
				class="h-96 w-96 rounded-lg transform-gpu pl-1"
				transition:slide={{
					delay: 250,
					duration: 300,
					easing: quintInOut,
					axis: 'x'
				}}
			>
				<Playlist {playlist} {loadTrack} {clearPlaylist} {currentTrackIndex} {removeSong} />
				<!-- Right Drawer Content -->
			</div>
		{:else}
			<div class="hidden" />
		{/if}
	</div>
	<Notification {message} />
	{#if isModalOpen}
		<ShortcutModal bind:isOpen={isModalOpen} />
	{/if}
</div>
