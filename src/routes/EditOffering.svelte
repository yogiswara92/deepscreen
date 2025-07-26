<script lang="ts">
  import { onMount } from 'svelte';

  export let params;

  let id = '';
  let offering = {
    nama_posisi: '',
    syarat: '',
    deskripsi: '',
    banner_url: ''
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
    max-width: 800px;
    margin: auto;
    padding: 2rem;
    margin-top:20px;
  }

  button {
    width: 200px;
    margin-right: 1rem;
  }

  button:disabled {
    background-color: #666;
    cursor: not-allowed;
  }
</style>

<div class="container">
  <h2>Edit Job Offering</h2>

  {#if loading}
    <p>Loading...</p>
  {:else}
    <label>Nama Posisi:</label>
    <input type="text" bind:value={offering.nama_posisi} />

    <label>Syarat:</label>
    <textarea rows="4" bind:value={offering.syarat}></textarea>

    <label>Deskripsi:</label>
    <textarea rows="4" bind:value={offering.deskripsi}></textarea>

    <label>Banner URL:</label>
    <input type="text" bind:value={offering.banner_url} />

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