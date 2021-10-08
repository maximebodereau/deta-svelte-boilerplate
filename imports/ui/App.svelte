<script>
  import { Deta } from "deta";
  import { onMount } from "svelte";
  import axios from "axios";
  import Resizer from "react-image-file-resizer";

  deta = Deta("DETAKEY");
  // Database
  prepstorant = deta.Base("BASE NAME");
  // Drive
  photosDrive = deta.Drive("DRIVE NAME");

  const headers = "X-API-Key: DETAKEY";

  //Key Name
  // Project id : DETA ID
  // https://database.deta.sh/v1/DETAID/BASE NAME

  let dataUrl;
  let file;
  const resizeFile = (file) =>
  new Promise((resolve) => {
    Resizer.imageFileResizer(
      file,
      300,
      300,
      "JPEG",
      100,
      0,
      (uri) => {
        resolve(uri);
      },
      "base64"
    );
  });

  async function handleFileLoaded(event) {
    // file = event.target.files[0];
    file = event.target.files[0];
    console.log(file);
    contentType = file.type;
    dataUrl = await resizeFile(file);
    refreshData();
  }

  // END FILE UPLOAD

  // FORM
  export let data = [];

  onMount(async () => {
    data = await prepstorant.fetch().then((x) => x.items);
  });

  async function refreshData() {
    data = await prepstorant.fetch().then((x) => x.items);
  }

  async function removeItem(id) {
    await prepstorant.delete(id);
    refreshData();
  }

  let newRestaurant = "";
  let newLocation = "";

  const handleSubmit = () => {
    prepstorant.insert({
      name: newRestaurant,
      createdAt: new Date(),
      location: newLocation,
      image: dataUrl,
    });

    // Clear form
    newRestaurant = "";
    newLocation = "";

    refreshData();
  };

  // END FORM
</script>

<!-- svelte-ignore missing-declaration -->
<div class="container-fluid" />

<form class="" on:submit|preventDefault={handleSubmit}>
  <input
    class="form-control mb-3"
    type="text"
    name="Restaurant"
    placeholder="Type to add new tasks"
    bind:value={newRestaurant}
  />
  <input
    class="form-control mb-3"
    type="text"
    name="Location"
    placeholder="Location"
    bind:value={newLocation}
  />

  <input
    class="block form-input mt-1 w-full"
    on:change={handleFileLoaded}
    placeholder="Select a data file"
    type="file"
  />

  <button class="btn btn-primary" type="submit">Add Restaurant</button>
</form>

<div class="col-12">
  <div class="row mt-3">
    {#each data as item}
      <div class="col-3 mb-3">
        <div class="card" style="width: 18rem;">
          <img src={item.image} class="card-img-top" alt="..." />
          <div class="card-body">
            <h5 class="card-title">{item.name}</h5>
            <p class="card-text">
              {item.location}
            </p>
            <button on:click={removeItem(item.key)} class="btn btn-danger"
              >delete</button
            >
          </div>
        </div>
      </div>
    {/each}
  </div>
</div>
