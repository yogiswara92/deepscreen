<script lang="ts">
  import { onMount } from 'svelte';

  export let params;

  let id = '';
  let offering = {
    nama_posisi: '',
    syarat: '',
    deskripsi: '',
    banner_url: '',
    start_date: '',
    end_date: '',
    is_active: '1'
  };
  let loading = true;
  let saving = false;
  let deleting = false;
  let email = '';

  onMount(async () => {
    id = params.id;
    email = localStorage.getItem('email') || '';
    if (!email) {
      alert("Please login first");
      window.location.href = '/';
      return;
    }

    const res = await fetch(`https://n8n.yesvara.com/webhook/ds-my-offering-detail?id=${id}&email=${encodeURIComponent(email)}`);
    if (res.ok) {
      const data = await res.json();
      offering = data[0];
      offering.is_active = String(offering.is_active);
    }
    loading = false;
  });

  async function saveChanges() {
    saving = true;

    const res = await fetch(`https://n8n.yesvara.com/webhook/ds-edit-offering`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        id,
        email,
        ...offering
      })
    });

    saving = false;

    if (res.ok) {
      alert('Offering berhasil diperbarui!');
      location.hash = `/myoffering`;
    } else {
      alert('Gagal memperbarui offering.');
    }
  }

  async function deleteOffering() {
    if (!confirm('Yakin ingin menghapus offering ini?')) return;
    deleting = true;

    const res = await fetch(`https://n8n.yesvara.com/webhook/ds-my-offering-delete?id=${id}&email=${encodeURIComponent(email)}`, {
      method: 'GET'
    });

    deleting = false;

    if (res.ok) {
      alert('Offering berhasil dihapus.');
      window.location.href = '/#/myoffering';
    } else {
      alert('Gagal menghapus offering.');
    }
  }
</script>

<style>
  .container {
    max-width: 600px;
    margin: auto;
    padding: 2rem;
    margin-top: 30px;
  }

  input, textarea, select {
    width: 100%;
    padding: 0.75rem;
    margin-bottom: 1rem;
    border-radius: 8px;
    border: none;
  }

  button {
    width: 200px;
    margin-right: 1rem;
  }

  button:disabled {
    background-color: #666;
    cursor: not-allowed;
  }

  .row {
    display: flex;
    gap: 1rem;
    flex-wrap: wrap;
    margin-bottom: 1rem;
  }

  .row > div {
    flex: 1 1 150px;
    min-width: 150px;
  }
</style>

<div class="container">
  <h2>Edit Job Offering</h2>

  {#if loading}
    <p>Loading...</p>
  {:else}
    <label>Position Name:</label>
    <input type="text" bind:value={offering.nama_posisi} />

    <label>Requirement:</label>
    <textarea rows="4" bind:value={offering.syarat}></textarea>

    <label>Description:</label>
    <textarea rows="4" bind:value={offering.deskripsi}></textarea>

    <div class="row">
      <div style="margin-right:10px">
        <label>Start Date:</label>
        <input type="date" bind:value={offering.start_date} />
      </div>
      <div style="margin-right:10px">
        <label>End Date:</label>
        <input type="date" bind:value={offering.end_date} />
      </div>
      <div>
        <label>Is Active:</label>
        <select bind:value={offering.is_active}>
          <option value="1">Yes</option>
          <option value="0">No</option>
        </select>
      </div>
    </div>

    <!-- <label>Banner URL:</label>
    <input type="text" bind:value={offering.banner_url} /> -->

    <div style="margin-top: 1rem">
      <button on:click={saveChanges} disabled={saving}>
        {saving ? 'Saving...' : 'Save Changes'}
      </button>
      <br><br>

      <button on:click={deleteOffering} disabled={deleting} style="background-color: red">
        {deleting ? 'Deleting...' : 'Delete Offering'}
      </button>
    </div>
  {/if}
</div>