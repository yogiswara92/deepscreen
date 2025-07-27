<script lang="ts">
    import { onMount } from 'svelte';
  
    let id_posisi = '';
    let email = '';
    let sessionDetails = [];
    let loading = true;
    export let params;
  
    let name = '';
    let jobPosition = '';
  
    onMount(async () => {
      const urlParams = new URLSearchParams(window.location.search);
      id_posisi = params.id;
      email = params.email;
      jobPosition = params.position;
  
      if (!id_posisi || !email) {
        alert('Missing required parameters');
        return;
      }
  
      const res = await fetch(`https://n8n.yesvara.com/webhook/ds-session-detail?id=${id_posisi}&email=${encodeURIComponent(email)}`);
      if (res.ok) {
        const data = await res.json();
        sessionDetails = data;
  
        if (data.length > 0) {
          const first = data[0];
          name = first.name;
          email = first.email;
        }
      } else {
        alert('Failed to fetch session data');
      }
      loading = false;
    });
  
    function formatDateTime(dt: string) {
      return new Date(dt).toLocaleString();
    }
  
    function formatScore(s) {
      const map = {
        DL: s.digital_leadership,
        GBS: s.global_business_savvy,
        CF: s.customer_focus,
        BSP: s.building_strategic_partnership,
        SO: s.strategic_orientation,
        DE: s.driving_execution,
        DI: s.driving_innovation,
        DOC: s.developing_organizational_capabilities,
        LC: s.leading_change,
        MD: s.managing_diversity,
      };
  
      return Object.entries(map)
        .filter(([_, val]) => val > 0)
        .map(([key, val]) => `${key}: ${val}`)
        .join(', ');
    }
  </script>
  
  <style>
    .konten {
      width: 90%;
      max-width: 900px;
      margin: auto;
      padding: 2rem;
    }
  
    .chat {
      display: flex;
      flex-direction: column;
      gap: 1rem;
      margin-top: 2rem;
    }
  
    .bubble {
      background: #f1f1f1;
      color: black;
      border-radius: 12px;
      padding: 1rem;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      text-align: justify;
    }
  
    .bubble h4 {
      margin: 0 0 0.5rem;
    }
  
    .meta {
      font-size: 0.8rem;
      color: #666;
      margin-top: 0.5rem;
      text-align: right;
    }
  </style>
  
  <div class="konten" style="margin-top:50px">
    <h2>{jobPosition}</h2>
  
    {#if loading}
      <p>Loading...</p>
    {:else}
      <div style="text-align: justify">
        <strong>Name:</strong> {name} <br>
        <strong>Email:</strong> {email}
      </div>
  
      <div class="chat">
        {#each sessionDetails as s}
          <div class="bubble">
            <p><strong>Q:</strong> {s.question}</p>
            <p><strong>A:</strong> {s.answer}</p>
            <p><strong>Insight:</strong> {s.insight}</p>
            {#if formatScore(s)}
                <p><strong>Detected Score:</strong> {formatScore(s)}</p>
            {:else}
                <p><strong>Detected Score:</strong> - </p>
            {/if}
            <div class="meta">{formatDateTime(s.created_at)}</div>
          </div>
        {/each}
      </div>
    {/if}
  </div>