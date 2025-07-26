<script lang="ts">
    // import { goto } from '$app/navigation';
  
    let name = '';
    let email = '';
    let position = '';
    let requirement = '';
    let description = '';
    let banner: File | null = null;
    let submitting = false;
  
    function handleFileChange(event: Event) {
      const input = event.target as HTMLInputElement;
      if (input?.files?.length) {
        banner = input.files[0];
      }
    }
  
    async function submitOffering() {
      if (!position || !requirement || !description) {
        alert('Fill in all the data first');
        return;
      }
  
      submitting = true;
      name = localStorage.getItem('nama') || '';
      email = localStorage.getItem('email') || '';
  
      const formData = new FormData();
      formData.append('name', name);
      formData.append('email', email);
      formData.append('position', position);
      formData.append('requirement', requirement);
      formData.append('description', description);
      formData.append('banner', banner);
  
      const res = await fetch('https://n8n.yesvara.com/webhook/ds-new-offering', {
        method: 'POST',
        body: formData
      });
  
      submitting = false;
  
      if (res.ok) {
        alert('Submit new job offering success!');
        location.hash = '#/myoffering';
      } else {
        alert('Submit failed!');
      }
    }
  </script>
  
  <style>
    .form-container {
      max-width: 600px;
      margin: auto;
      padding: 2rem;
    }
  
    input, textarea, select {
      width: 100%;
      padding: 0.75rem;
      margin-bottom: 1rem;
      border-radius: 8px;
      border: none;
    }
  
    button {
      background-color: #01a3a4;
      color: white;
      padding: 1rem 2rem;
      border: none;
      border-radius: 8px;
      cursor: pointer;
    }
  
    button:disabled {
      background-color: #666;
      cursor: not-allowed;
    }
  </style>
  
  <div class="form-container">
    <h2 style="margin-top:50px">New Job Offering</h2>
  
    <label>Position Name:</label>
    <input type="text" bind:value={position} placeholder="Position Name" required />
    <label>Requirement:</label>
    <textarea rows="4" bind:value={requirement} placeholder="Job Requirement" required></textarea>
    <label>Description:</label>
    <textarea rows="4" bind:value={description} placeholder="Job Description" required></textarea>
  
    <label>Image Upload:</label>
    <input type="file" accept="image/*" on:change={handleFileChange} required />
  
    <button on:click={submitOffering} disabled={submitting}>
      {submitting ? 'Submitting...' : 'Submit'}
    </button>
  </div>