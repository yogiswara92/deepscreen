<script lang="ts">
    let name = '';
    let email = '';
    let position = '';
    let requirement = '';
    let description = '';
    let banner: File | null = null;
    let start_date = '';
    let end_date = '';
    let is_active = '1';
    let submitting = false;
  
    function handleFileChange(event: Event) {
      const input = event.target as HTMLInputElement;
      if (input?.files?.length) {
        banner = input.files[0];
      }
    }
  
    async function submitOffering() {
      if (!position || !requirement || !description || !start_date || !end_date) {
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
      formData.append('start_date', start_date);
      formData.append('end_date', end_date);
      formData.append('is_active', is_active);
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
  
  <div class="form-container">
    <h2 style="margin-top:50px">New Job Offering</h2>
  
    <label>Position Name:</label>
    <input type="text" bind:value={position} placeholder="Position Name" required />
  
    <label>Requirement:</label>
    <textarea rows="4" bind:value={requirement} placeholder="Job Requirement" required></textarea>
  
    <label>Description:</label>
    <textarea rows="4" bind:value={description} placeholder="Job Description" required></textarea>
  
    <div class="row">
      <div style="margin-right:10px">
        <label>Start Date:</label>
        <input type="date" bind:value={start_date} required />
      </div>
      <div style="margin-right:10px">
        <label>End Date:</label>
        <input type="date" bind:value={end_date} required />
      </div>
      <div>
        <label>Is Active:</label>
        <select bind:value={is_active}>
          <option value="1">Yes</option>
          <option value="0">No</option>
        </select>
      </div>
    </div>
  
    <!-- <label>Image Upload:</label>
    <input type="file" accept="image/*" on:change={handleFileChange} required /> -->
  
    <button on:click={submitOffering} disabled={submitting}>
      {submitting ? 'Submitting...' : 'Submit'}
    </button>
  </div>