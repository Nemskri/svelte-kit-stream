<script>
	let mediaRecorder;
	let isRecording = false;
	let audioLevel = 0;
	let animationFrame = 0;
	
	// Animation interval reference
	let animationInterval;
	
	// Update animation when recording state changes
	$: if (isRecording) {
		startAnimation();
	} else {
		stopAnimation();
	}
	
	function startAnimation() {
		animationInterval = setInterval(() => {
			animationFrame = (animationFrame + 1) % 100;
			// Simulate audio levels when recording
			audioLevel = Math.random() * 0.5 + 0.5;
		}, 100);
	}
	
	function stopAnimation() {
		if (animationInterval) {
			clearInterval(animationInterval);
			audioLevel = 0;
		}
	}

	async function startRecording() {
		try {
			const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
			mediaRecorder = new MediaRecorder(stream);
			
			mediaRecorder.ondataavailable = async (e) => {
				if (e.data.size > 0) {
					const audioBlob = e.data;
					// Next step: send this to FastAPI via WebSocket
					console.log('Audio blob:', audioBlob);
				}
			};
			
			mediaRecorder.start();
			isRecording = true;
		} catch (error) {
			console.error("Error accessing microphone:", error);
		}
	}

	function stopRecording() {
		if (mediaRecorder) {
			mediaRecorder.stop();
			// Stop all audio tracks
			mediaRecorder.stream.getTracks().forEach(track => track.stop());
			isRecording = false;
		}
	}

	function toggleRecording() {
		if (isRecording) {
			stopRecording();
		} else {
			startRecording();
		}
	}
</script>

<div class="flex flex-col items-center justify-center w-full min-h-screen bg-gradient-to-br from-indigo-900 to-purple-900 px-6">
	<div class="w-full max-w-md bg-white/10 backdrop-blur-lg rounded-2xl shadow-2xl overflow-hidden border border-white/20">
		<div class="flex flex-col items-center p-8">
			<h1 class="text-2xl font-bold text-white mb-2">AI Voice Assistant</h1>
			<p class="text-purple-200 mb-8 text-center">Speak naturally and receive AI responses in real-time</p>
			
			<!-- Voice visualization -->
			<div class="w-full h-24 mb-8 rounded-xl bg-black/20 flex items-center justify-center overflow-hidden">
				{#if isRecording}
					<div class="flex items-end justify-center w-full h-full px-4">
						{#each Array(20) as _, i}
							{@const intensity = 0.3 + (Math.sin((i + animationFrame) * 0.3) + 1) * 0.5 * audioLevel}
							<div 
								class="w-2 mx-1 bg-gradient-to-t from-purple-500 to-indigo-300 rounded-t-full"
								style="height: {intensity * 100}%;"
							></div>
						{/each}
					</div>
				{:else}
					<p class="text-purple-200 text-opacity-60">Ready to listen...</p>
				{/if}
			</div>
			
			<!-- Recording button -->
			<button 
				on:click={toggleRecording}
				class="group relative flex items-center justify-center w-20 h-20 rounded-full transition-all duration-300 ease-in-out focus:outline-none"
			>
				<div class={`absolute inset-0 rounded-full transition-all duration-300 ${isRecording ? 'bg-red-500 animate-pulse' : 'bg-indigo-600 hover:bg-indigo-500'}`}></div>
				<div class="absolute inset-1 rounded-full bg-gradient-to-br from-white/20 to-transparent"></div>
				
				{#if isRecording}
					<!-- Stop icon -->
					<div class="relative w-6 h-6 bg-white rounded-sm"></div>
				{:else}
					<!-- Mic icon -->
					<svg xmlns="http://www.w3.org/2000/svg" class="relative w-8 h-8 text-white" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
						<path d="M12 1a3 3 0 0 0-3 3v8a3 3 0 0 0 6 0V4a3 3 0 0 0-3-3z"></path>
						<path d="M19 10v2a7 7 0 0 1-14 0v-2"></path>
						<line x1="12" y1="19" x2="12" y2="23"></line>
						<line x1="8" y1="23" x2="16" y2="23"></line>
					</svg>
				{/if}
			</button>
			
			<p class="mt-6 text-white font-medium">
				{isRecording ? 'Tap to stop' : 'Tap to speak'}
			</p>
			
			{#if isRecording}
				<div class="mt-8 px-6 py-3 bg-white/10 rounded-lg border border-white/20">
					<p class="text-white text-sm">Listening to your voice...</p>
				</div>
			{/if}
		</div>
	</div>
	
	<p class="mt-8 text-purple-200 text-opacity-80 text-sm">Powered by Elevate AI Voice Technology</p>
</div>