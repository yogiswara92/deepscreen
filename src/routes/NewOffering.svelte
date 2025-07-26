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
        width:100%;
      max-width: 800px;
      /* width:100%; */
      margin: auto;
      padding: 2rem;
    }
  
    button {
      background-color: #01a3a4;
      color: white;
      padding: 1rem 2rem;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      width:100%;
      max-width:700px;
    }
  
    button:disabled {
      background-color: #666;
      cursor: not-allowed;
    }

    input, textarea, select{
        width:90%;
        max-width:700px;
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
  
    <div>
        <label>Position Name:</label> <br />
        <input type="text" bind:value={position} placeholder="Position Name" required />
    </div>

    <div>
        <label>Requirement:</label> <br />
        <textarea rows="4" bind:value={requirement} placeholder="Job Requirement" required></textarea>
    </div>

    <div>
        <label>Description:</label> <br />
        <textarea rows="4" bind:value={description} placeholder="Job Description" required></textarea>
    </div>
    
    <!-- <div class="row"> -->
      <div > 
        <label>Start Date:</label><br />
        <input type="date" bind:value={start_date} required />
      </div>
      <div >
        <label>End Date:</label><br />
        <input type="date" bind:value={end_date} required />
      </div>
      <div>
        <label>Is Active:</label> <br />
        <select bind:value={is_active}>
          <option value="1">Yes</option>
          <option value="0">No</option>
        </select>
      </div>
    <!-- </div> -->
  
    <!-- <label>Image Upload:</label>
    <input type="file" accept="image/*" on:change={handleFileChange} required /> -->
  
    <button on:click={submitOffering} disabled={submitting}>
      {submitting ? 'Submitting...' : 'Submit'}
    </button>
  </div>