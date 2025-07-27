<script lang="ts">
  import { onMount } from 'svelte';
  export let params;

  let id = '';
  let email = '';
  let nama_posisi='';
  let offering: any = {};
  let participants: any[] = [];
  let loading = true;
  let isMobile = false;

  onMount(async () => {
    id = params.id;
    email = localStorage.getItem('email') || '';
    if (!email) {
      alert("Please login first");
      location.href = '/';
      return;
    }

    isMobile = window.innerWidth < 768;
    window.addEventListener('resize', () => {
      isMobile = window.innerWidth < 768;
    });

    // Fetch offering detail
    const resOffering = await fetch(`https://n8n.yesvara.com/webhook/ds-my-offering-detail?id=${id}&email=${encodeURIComponent(email)}`);
    if (resOffering.ok) {
      const data = await resOffering.json();
      offering = data[0];
      nama_posisi=offering.nama_posisi;
    }

    // Fetch participants + score
    const resParticipants = await fetch(`https://n8n.yesvara.com/webhook/ds-session-list?id=${id}&email=${encodeURIComponent(email)}`);
    if (resParticipants.ok) {
      participants = await resParticipants.json();
    }

    loading = false;
  });

  function goToEdit() {
      location.hash = `/editoffering/${id}`;
  }

  function goToSession(email: string) {
      location.hash = `/detailsession/${id}/${nama_posisi}/${email}`;
  }
</script>

<style>
.konten {
  width:90%;  
  max-width: 900px;
  margin: auto;
  padding: 2rem;
}
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.participant-table {
  width: 100%;
  margin-top: 2rem;
  border-collapse: collapse;
}
.participant-table th, .participant-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
}
.button-ico{
    width:20px;
    padding-left:13px;
    padding-top:7px;
    height:30px;
}

.skor{
  width:100%;
  border:0px;
}
.skor td{
  padding:0px; border:0px;
  text-align:left;
  width:10px;
}
</style>

<div class="konten" style="margin-top:50px">
{#if loading}
  <p>Loading...</p>
{:else}
  <div class="header">
    <h2>{offering.nama_posisi}</h2>
    <button on:click={goToEdit} class="button-ico"><i class="fas fa-edit"></i></button>
  </div>

  <div style="text-align: justify">
    <u><strong>Job Description:</strong></u><br /><div style="white-space: pre-line;">{offering.deskripsi}</div>
  </div>
  <br>
  <div style="text-align: justify">
    <u><strong>Job Requirement:</strong></u><br /><div style="white-space: pre-line;">{offering.syarat}</div>
  </div>
  <br>
  <div style="text-align: justify">
    <strong>Start Date:</strong> {offering.start_date?.split('T')[0]} <br>
    <strong>End Date:</strong> {offering.end_date?.split('T')[0]}<br>
    <strong>Status:</strong> <b style="color:{offering.is_active == 1 ? 'green': 'red'}">{offering.is_active == 1 ? 'Active': 'Not Active'}</b>
  </div>

  <hr />
  <h3>Interview Results</h3>
  {#if !participants[0]?.id }
    <p>No participants have completed the interview yet.</p>
  {:else}
    <table class="participant-table">
      <thead>
        <tr>
          <th>Name</th>
          {#if !isMobile}
            <th>DL</th>
            <th>GBS</th>
            <th>CF</th>
            <th>BSP</th>
            <th>SO</th>
            <th>DE</th>
            <th>DI</th>
            <th>DOC</th>
            <th>LC</th>
            <th>MD</th>
          {:else}
            <th>Score</th>
          {/if}
          <th>Action</th>
        </tr>
      </thead>
      <tbody>
        {#each participants as p}
          <tr>
            <td>{p.name} <div style="font-size:10px">{p.email}</div></td>
            {#if !isMobile}
              <td>{p.dl}</td>
              <td>{p.gbs}</td>
              <td>{p.cf}</td>
              <td>{p.bsp}</td>
              <td>{p.so}</td>
              <td>{p.de}</td>
              <td>{p.di}</td>
              <td>{p.doc}</td>
              <td>{p.lc}</td>
              <td>{p.md}</td>
            {:else}
              <td style="text-align:left; font-size:12px">
                <table class="skor">
                {#if p.dl}<tr><td>DL</td><td>:</td><td>{p.dl}</td></tr>{/if}
                {#if p.gbs}<tr><td>GBS</td><td>:</td><td>{p.gbs}</td></tr>{/if}
                {#if p.cf}<tr><td>CF</td><td>:</td><td>{p.cf}</td></tr>{/if}
                {#if p.bsp}<tr><td>BSP</td><td>:</td><td>{p.bsp}</td></tr>{/if}
                {#if p.so}<tr><td>SO</td><td>:</td><td>{p.so}</td></tr>{/if}
                {#if p.de}<tr><td>DE</td><td>:</td><td>{p.de}</td></tr>{/if}
                {#if p.di}<tr><td>DI</td><td>:</td><td>{p.di}</td></tr>{/if}
                {#if p.doc}<tr><td>DOC</td><td>:</td><td>{p.doc}</td></tr>{/if}
                {#if p.lc}<tr><td>LC</td><td>:</td><td>{p.lc}</td></tr>{/if}
                {#if p.md}<tr><td>MD</td><td>:</td><td>{p.md}</td></tr>{/if}
              </table>
              </td>
            {/if}
            <td>
              <button on:click={() => goToSession(p.email)} class="button-ico"><i class="fas fa-magnifying-glass"></i></button>
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  {/if}
{/if}
</div>