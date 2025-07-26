<script lang="ts">
    import { onMount } from 'svelte';
    // import { goto } from '$app/navigation';
  
    let email = '';
    let offerings = [];
    let loading = true;
  
    onMount(async () => {
      email = localStorage.getItem('email') || '';
      if (!email) {
        alert("Please login first");
        window.location.href = '/';
        return;
      }
  
      const res = await fetch(`https://n8n.yesvara.com/webhook/ds-get-all-offering`);
      if (res.ok) {
        offerings = await res.json();
        // console.log(offerings);
      } else {
        offerings = [];
      }
      loading = false;
    });
  
    function goToDetail(id: string) {
        location.hash = `/interview/${id}`;
    }
  
  </script>
  
  <style>
    .container {
      max-width: 800px;
      margin: auto;
      padding: 2rem;
    }
  
    .card {
      background: #ffffff10;
      padding: 1rem;
      border: 1px solid #ffffff30;
      border-radius: 10px;
      margin-top:0.5rem;
      margin-bottom: 0.5rem;
      cursor: pointer;
      width:400px;
      max-width:99%
    }
  
    .card:hover {
      background-color: #01313e;
    }
  
    h1 {
      margin-bottom: 1rem;
    }
  </style>
  
  <div class="container">
    <h2 style="margin-top:50px">Available Session</h2>
  
    {#if loading}
      <p>Loading...</p>
    {:else if offerings.length === 0}
      <p>You don't have any job offerings.</p>
    {:else}
      {#each offerings as job}
        <div class="card" on:click={() => goToDetail(job.id)}>
          <b>{job.nama_posisi}</b>
          <p style="font-size:10px">{job.deskripsi}</p>
        </div>
      {/each}
    {/if}
  </div>