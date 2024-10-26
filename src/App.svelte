<script>
  let patients = [];
  let searchQuery = "";
  let newPatient = { name: "", gender: "", birthDate: "", phone: "" };
  let selectedPatient = null;

  // Temporary form values
  let tempGivenName = "";
  let tempGender = "";
  let tempBirthDate = "";
  let tempPhone = "";

  // FHIR server URL
  const fhirServer = "https://fhir-bootcamp.medblocks.com/fhir/Patient";

  // Fetch list of patients from FHIR server
  async function listPatients() {
    const response = await fetch(`${fhirServer}`);
    const data = await response.json();

    // Extract the resources (patients) from the FHIR server response
    patients = data.entry.map(entry => entry.resource);

    // Check if patients have given names and handle any missing data
    patients.forEach(patient => {
      if (!patient.name & !patient.name.length & !patient.name[0].given & !patient.name[0].given.length) {
        patient.name = [{ given: ["Unknown"] }]; // Fallback to "Unknown" if no given name is present
      }
    });
  }

  // Add a new patient to the FHIR server
  async function addPatient() {
    if (!tempGivenName || !tempGender || !tempBirthDate || !tempPhone) {
      alert("Please fill all fields.");
      return;
    }

    const response = await fetch(fhirServer, {
      method: "POST",
      headers: {
        "Content-Type": "application/fhir+json",
      },
      body: JSON.stringify({
        resourceType: "Patient",
        name: [{ given: [tempGivenName] }],
        gender: tempGender,
        birthDate: tempBirthDate,
        telecom: [{ system: "phone", value: tempPhone }],
      }),
    });

    if (response.ok) {
      alert("Patient added successfully.");
      await listPatients();
      tempGivenName = tempGender = tempBirthDate = tempPhone = ""; // Reset form
    } else {
      alert("Failed to add patient.");
    }
  }

  // Update an existing patient
  async function updatePatient() {
    selectedPatient.name[0].given[0] = tempGivenName;
    selectedPatient.gender = tempGender;
    selectedPatient.birthDate = tempBirthDate;
    selectedPatient.telecom[0].value = tempPhone;

    const response = await fetch(`${fhirServer}/${selectedPatient.id}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/fhir+json",
      },
      body: JSON.stringify(selectedPatient),
    });

    if (response.ok) {
      alert("Patient updated successfully.");
      await listPatients();
      selectedPatient = null; // Clear selection
      tempGivenName = tempGender = tempBirthDate = tempPhone = ""; // Reset form
    } else {
      alert("Failed to update patient.");
    }
  }

  // Populate form for editing
  function editPatient(patient) {
    selectedPatient = patient;
    tempGivenName = patient.name[0]?.given[0] || "";
    tempGender = patient.gender || "";
    tempBirthDate = patient.birthDate || "";
    tempPhone = patient.telecom[0]?.value || "";
  }

  // Search patients by name or phone number
  // async function searchPatients() {
  //   const query = searchQuery ? `?name=${searchQuery}&telecom=${searchQuery}` : "";
  //   const response = await fetch(`${fhirServer}${query}`);
  //   const data = await response.json();
  //   patients = data.entry ? data.entry.map(entry => entry.resource) : [];
  // }
  async function searchPatients() {
    let query = "";

    if (searchQuery) {
      // Check if the search query looks like a phone number (contains only digits, spaces, or dashes)
      const isPhoneNumber = /^\d[\d-\s]+$/.test(searchQuery);

      if (isPhoneNumber) {
        // Search by phone number
        query = `?telecom=${searchQuery}`;
      } else {
        // Search by given name
        query = `?given=${searchQuery}`;
      }
    }

    const response = await fetch(`${fhirServer}${query}`);
    const data = await response.json();
    patients = data.entry ? data.entry.map(entry => entry.resource) : [];
  }

  // Load patients on component mount
  listPatients();
</script>

<div class="p-4">
  <!-- Search Form -->
  <div class="mb-4">
    <input
      type="text"
      class="border p-2 rounded w-full"
      bind:value={searchQuery}
      placeholder="Search by name or phone number..."
    />
    <button
      class="bg-blue-500 text-white p-2 mt-2 rounded"
      on:click={searchPatients}
    >
      Search
    </button>
  </div>

  <!-- Patient List in Table Format (Only Given Names) -->
  <div>
    <h2 class="text-xl mb-4">Patients List</h2>
    <table class="min-w-full table-auto border-collapse border border-gray-400">
      <thead>
        <tr class="bg-gray-200">
          <th class="border border-gray-400 px-4 py-2">Given Name</th>
          <th class="border border-gray-400 px-4 py-2">Gender</th>
          <th class="border border-gray-400 px-4 py-2">Date of Birth</th>
          <th class="border border-gray-400 px-4 py-2">Actions</th>
        </tr>
      </thead>
      <tbody>
        {#each patients as patient}
          <tr>
            <td class="border border-gray-400 px-4 py-2">{patient.name[0]?.given[0]}</td>
            <td class="border border-gray-400 px-4 py-2">{patient.gender}</td>
            <td class="border border-gray-400 px-4 py-2">{patient.birthDate}</td>
            <td class="border border-gray-400 px-4 py-2">
              <button
                class="text-blue-500 underline"
                on:click={() => editPatient(patient)}
              >
                Edit
              </button>
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>

  <!-- Patient Form -->
  <div class="mt-8">
    <h2 class="text-xl mb-4">Add / Edit Patient</h2>
    <form on:submit|preventDefault={selectedPatient ? updatePatient : addPatient}>
      <input
        type="text"
        class="border p-2 mb-2 w-full"
        bind:value={tempGivenName}
        placeholder="Given Name"
      />
      <input
        type="text"
        class="border p-2 mb-2 w-full"
        bind:value={tempGender}
        placeholder="Gender"
      />
      <input
        type="date"
        class="border p-2 mb-2 w-full"
        bind:value={tempBirthDate}
      />
      <input
        type="text"
        class="border p-2 mb-2 w-full"
        bind:value={tempPhone}
        placeholder="Phone Number"
      />
      <button class="bg-green-500 text-white p-2 rounded">
        {selectedPatient ? "Update Patient" : "Add Patient"}
      </button>
    </form>
  </div>
</div>





