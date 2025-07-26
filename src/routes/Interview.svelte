<script lang="ts">
  import { onMount } from 'svelte';

  export let params;

  let id = '';

  let recording = false;
  let mediaRecorder: MediaRecorder;
  let audioChunks: Blob[] = [];
  let audioBlob: Blob | null = null;
  let timer = 0;
  let timerInterval: NodeJS.Timer;
  let transcript = '';
  let name = '';
  let email='';
  let nameLocked = false;
  let isFinished=false;
  let model = "model1";
  let buttonLock=true;
  // Array posisi pekerjaan
  const jobs = { id:0, position: "", requirements: "", description:"" };
  let syarat = jobs.requirements;
  let job_position =jobs.position;
  let id_posisi=jobs.id;
  let deskripsi="";

  let question ="";

  let recognition: SpeechRecognition;
  const SpeechRecognition = window.SpeechRecognition || (window as any).webkitSpeechRecognition;

  // Scroll ke bawah setiap kali jawaban berubah
  let textareaEl: HTMLTextAreaElement | null = null;
  $: if (textareaEl && transcript) {
    textareaEl.scrollTop = textareaEl.scrollHeight;
  }

  onMount(async () => {
    if(!localStorage.getItem('access_token')){
      alert("Please login first");
      window.location.href = '/';
    }
    id = params.id;
    name=localStorage.getItem('nama');
    email=localStorage.getItem('email');

    const res = await fetch(`https://n8n.yesvara.com/webhook/ds-detail-offering?id=${id}`);
    if (res.ok) {
      const data = await res.json();
      id_posisi = data[0].id;
      job_position= data[0].nama_posisi; 
      syarat= data[0].syarat; 
      deskripsi=data[0].deskripsi;
      //  console.log(data[0]);
    }   

    if (SpeechRecognition) {
      recognition = new SpeechRecognition();
      recognition.continuous = true;
      recognition.interimResults = true;
      recognition.lang = 'id-ID';
 
      recognition.onresult = (event: SpeechRecognitionEvent) => {
        const result = Array.from(event.results)
          .map(res => res[0].transcript)
          .join('');
        transcript = result;
      };
      // speakQuestion(question);
      recognition.onerror = e => console.error('Recognition error:', e);
    }
  });

  function handleJobChange(event) {
    const selected = jobs.find(j => j.id === +event.target.value);
    job_position = selected.position;
    syarat = selected.requirements;
    id_posisi = selected.id;
  
  }

  function startInterview(){
    
    nameLocked = true;
    question = "Hi "+name+", salam kenal. Boleh perkenalan dan menceritakan pengalaman anda yang berkaitan dengan "+job_position+"?";
  }

  async function startRecording() {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    mediaRecorder = new MediaRecorder(stream);

    mediaRecorder.ondataavailable = event => {
      audioChunks.push(event.data);
    };

    mediaRecorder.onstop = () => {
      audioBlob = new Blob(audioChunks, { type: 'audio/webm' });
    };

    audioChunks = [];
    mediaRecorder.start();
    if (recognition) recognition.start();
    timer = 0;
    timerInterval = setInterval(() => timer++, 1000);
    recording = true;
  }

  function stopRecording() {
    mediaRecorder.stop();
    if (recognition) recognition.stop();
    clearInterval(timerInterval);
    recording = false;
    buttonLock=false;
  }

  async function sendAudio() {
    if (!audioBlob) return;

    buttonLock=true;

    const formData = new FormData();
    formData.append('name', name);
    formData.append('email', email);
    formData.append("model", model);
    // formData.append('audio', audioBlob, 'interview.webm');
    formData.append('timestamp', new Date().toISOString());
    formData.append('question', question);
    formData.append('transcript', transcript);
    formData.append('syarat', syarat);
    formData.append('job_position', job_position);
    formData.append('id_posisi', id);
    
    //production: https://n8n.yesvara.com/webhook/interview-submit
    //testing: https://n8n.yesvara.com/webhook-test/interview-submit
    const res=await fetch('https://n8n.yesvara.com/webhook/ds-interview', {
      method: 'POST',
      body: formData
    });

    //nameLocked = true;
    alert('Jawaban berhasil dikirim');

    const data = await res.json();

    if (data){
      // console.log("response:", data);
    }
    // ✅ Update pertanyaan berikutnya
    const nextQuestion = data[0]?.next_question;

    if (nextQuestion) {
      question = nextQuestion; 
      buttonLock=true;
      // console.log("next_question:", data[0]?.next_question);
      transcript='';
    }

    if(data[0]?.isFinished){
      isFinished=true;
    }

    
  }

  function speakQuestion(text: string) {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = 'id-ID'; // Bahasa Indonesia
    utterance.rate = 1.3;       // Kecepatan bicara (0.1 – 10)
    speechSynthesis.speak(utterance);
  }

 
</script>

<svelte:head>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css"
    integrity="sha512-..."
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
    font-size: 1.5rem;
    font-size:20px;
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

  .transcript {
    /* margin-top: 2rem; */
    background: #ffffff10;
    padding: 1rem;
    border: 1px solid #ffffff30;
    border-radius: 10px;
    width:90%;
    max-width: 600px;
    margin-bottom:15px;
  }

  textarea.editable-text {
    border: none;
    resize: vertical;
    background-color: transparent;
    color: white;
    width: 100%;
    max-width: 600px; 
    outline: none;
  }
</style>

<div class="container" style="margin-top:30px ">
{#if !nameLocked}
   
    <h2>Interview Session</h2>

    {#if job_position}
      <div style="text-align: justify; width:100%">
        <u><b>Job Position:</b></u>
        {job_position}
      </div>
      
      <br>
      
      <div style="text-align: justify">
        <u><b>Job Description:</b></u>
        <div style="white-space: pre-line;">{deskripsi}</div>
      </div>
      <br>

      <div style="text-align: justify">
        <u><b>Job Requirement:</b></u>
        <div style="white-space: pre-line;">{syarat}</div>
      </div>

      <br>

      <div class="input-nama" >
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
        <button on:click={startInterview} style="width:250px; margin-top:10px; padding-left:60px" >
          <i class="fas fa-paper-plane"></i> Start Interview
        </button>
      </div>
    {:else}
      <i>Loading data..</i>
    {/if}
  {:else}

  <div class="question" style="margin-top:10vh">    
    {question}
  </div>

  {#if recording}
    <div class="timer">Durasi: {timer} detik</div>
  {/if}

  {#if transcript}
    <div class="transcript" >
      <strong>Jawaban anda:</strong><br />
      <textarea
        id="transcript"
        bind:this={textareaEl}
        bind:value={transcript}
        rows="3"
        class="editable-text"
      ></textarea>
    </div>
  {/if}  

  {#if !isFinished}
    <div class="controls">
      {#if !recording}
        <button on:click={startRecording}>
          <i class="fas fa-microphone"></i> Jawab
        </button>
      {:else}
        <button on:click={stopRecording} style="background-color:red">
          <i class="fas fa-stop"></i> Cukup
        </button>
      {/if}
      <button on:click={sendAudio} disabled={buttonLock}>
        <i class="fas fa-paper-plane"></i> Kirim
      </button>
    </div>
  {/if}
{/if} 
</div>