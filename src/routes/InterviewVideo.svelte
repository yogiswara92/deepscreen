<script lang="ts">
    import { onMount } from 'svelte';
  
    export let params;
  
    let id = '';
    let recording = false;
    let mediaRecorder: MediaRecorder;
    let videoChunks: Blob[] = [];
    let videoBlob: Blob | null = null;
    let timer = 0;
    let timerInterval: NodeJS.Timer;
    let name = '';
    let email = '';
    let nameLocked = false;
    let isFinished = false;
    let model = "model1";
    let buttonLock = true;
    const jobs = { id: 0, position: "", requirements: "", description: "" };
    let syarat = jobs.requirements;
    let job_position = jobs.position;
    let id_posisi = jobs.id;
    let deskripsi = "";
  
    let question = "";
  
    let videoStream: MediaStream;
    let videoElement: HTMLVideoElement;
  
    onMount(async () => {
      if (!localStorage.getItem('access_token')) {
        alert("Please login first");
        window.location.href = '/';
      }
      id = params.id;
      name = localStorage.getItem('nama');
      email = localStorage.getItem('email');
  
      const res = await fetch(`https://n8n.yesvara.com/webhook/ds-detail-offering?id=${id}`);
      if (res.ok) {
        const data = await res.json();
        id_posisi = data[0].id;
        job_position = data[0].nama_posisi;
        syarat = data[0].syarat;
        deskripsi = data[0].deskripsi;
      }
    });
  
    function startInterview() {
      nameLocked = true;
      question = "Hi " + name + ", salam kenal. Boleh perkenalan dan menceritakan pengalaman anda yang berkaitan dengan " + job_position + "?";
    }
  
    async function startRecording() {
      videoStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
      videoElement.srcObject = videoStream;
      videoElement.play();
  
      mediaRecorder = new MediaRecorder(videoStream);
  
      mediaRecorder.ondataavailable = event => {
        videoChunks.push(event.data);
      };
  
      mediaRecorder.onstop = () => {
        videoBlob = new Blob(videoChunks, { type: 'video/webm' });
      };
  
      videoChunks = [];
      mediaRecorder.start();
      timer = 0;
      timerInterval = setInterval(() => timer++, 1000);
      recording = true;
    }
  
    function stopRecording() {
      mediaRecorder.stop();
      clearInterval(timerInterval);
      recording = false;
      buttonLock = false;
    }
  
    async function sendVideo() {
      if (!videoBlob) return;
  
      buttonLock = true;
  
      const formData = new FormData();
      formData.append('name', name);
      formData.append('email', email);
      formData.append("model", model);
      formData.append('timestamp', new Date().toISOString());
      formData.append('question', question);
      formData.append('syarat', syarat);
      formData.append('job_position', job_position);
      formData.append('id_posisi', id);
      formData.append('video', videoBlob, 'interview.webm');
  
      const res = await fetch('https://n8n.yesvara.com/webhook/ds-video-interview', {
        method: 'POST',
        body: formData
      });
  
      alert('Jawaban berhasil dikirim');
      const data = await res.json();
  
      if (data[0]?.next_question) {
        question = data[0].next_question;
        buttonLock = true;
      }
  
      if (data[0]?.isFinished) {
        isFinished = true;
      }
    }
  </script>
  
  <svelte:head>
    <link
      rel="stylesheet"
      href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css"
      crossorigin="anonymous"
      referrerpolicy="no-referrer"
    />
  </svelte:head>
  
  <style>
    :global(body) {
      margin: 0;
      font-family: 'Segoe UI', sans-serif;
      background-color: #01212e;
      color: white;
    }
    .container {
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 1rem;
      text-align: center;
      justify-content: center;
    }
    .question {
      font-size: 20px;
      margin-bottom: 2rem;
      max-width: 600px;
    }
    .controls {
      display: flex;
      gap: 1rem;
      flex-wrap: wrap;
      justify-content: center;
    }
    button {
      background-color: #01a3a4;
      border: none;
      padding: 1rem 2rem;
      color: white;
      font-size: 1rem;
      width: 130px;
      border-radius: 8px;
      cursor: pointer;
      transition: background 0.3s ease;
      display: flex;
      align-items: center;
      gap: 0.5rem;
    }
    button:hover {
      background-color: #00cec9;
    }
    button:disabled {
      background-color: #555;
      cursor: not-allowed;
    }
    .timer {
      margin-top: 1rem;
      font-size: 1.2rem;
    }
    video {
      width: 90%;
      max-width: 600px;
      border: 2px solid white;
      border-radius: 10px;
      margin-bottom: 1rem;
    }
  </style>
  
  <div class="container" style="margin-top:30px">
    {#if !nameLocked}
      <h2>Interview Session</h2>
      {#if job_position}
        <div style="text-align: justify; width:100%">
          <u><b>Job Position:</b></u> {job_position}
        </div>
        <br>
        <div style="text-align: justify; width:100%">
          <u><b>Job Description:</b></u>
          <div style="white-space: pre-line;">{deskripsi}</div>
        </div>
        <br>
        <div style="text-align: justify; width:100%">
          <u><b>Job Requirement:</b></u>
          <div style="white-space: pre-line;">{syarat}</div>
        </div>
        <br>
        <div class="input-nama">
          <label for="name">Your Name:</label><br />
          <input
            id="name"
            type="text"
            bind:value={name}
            required
            style="padding: 0.5rem; border-radius: 8px; border: none; width: 230px; height:40px"
            placeholder="Tuliskan nama Anda disini"
          />
        </div>
        <div class="controls">
          <button on:click={startInterview} style="width:250px; margin-top:10px; padding-left:60px">
            <i class="fas fa-paper-plane"></i> Start Interview
          </button>
        </div>
      {:else}
        <i>Loading data..</i>
      {/if}
    {:else}
      <div class="question" style="margin-top:10vh">{question}</div>
      <video bind:this={videoElement} autoplay playsinline muted></video>
      {#if recording}
        <div class="timer">Durasi: {timer} detik</div>
      {/if}
      {#if !isFinished}
        <div class="controls">
          {#if !recording}
            <button on:click={startRecording}>
              <i class="fas fa-video"></i> Rekam
            </button>
          {:else}
            <button on:click={stopRecording} style="background-color:red">
              <i class="fas fa-stop"></i> Stop
            </button>
          {/if}
          <button on:click={sendVideo} disabled={buttonLock}>
            <i class="fas fa-paper-plane"></i> Kirim
          </button>
        </div>
      {/if}
    {/if}
  </div>