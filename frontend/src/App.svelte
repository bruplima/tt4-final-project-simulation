<script>
  import { onMount } from "svelte";

  let users = [];
  let newUser = { title: "", description: "", completed: false };
  let editIndex = -1;

  const baseUrl = "http://localhost:8080/api/users"; // Use "http://api:8080/api/users" se for dentro do Docker

  async function fetchUsers() {
    const res = await fetch(baseUrl);
    users = await res.json();
  }

  async function addUser() {
    const res = await fetch(baseUrl, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(newUser),
    });
    if (res.ok) {
      newUser = { title: "", description: "", completed: false };
      await fetchUsers();
    }
  }

  async function deleteUser(id) {
    await fetch(`${baseUrl}/${id}`, { method: "DELETE" });
    await fetchUsers();
  }

  function startEdit(index) {
    editIndex = index;
  }

  async function saveEdit(user) {
    await fetch(`${baseUrl}/${user.id}`, {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(user),
    });
    editIndex = -1;
    await fetchUsers();
  }

  onMount(fetchUsers);
</script>

<svelte:head>
  <link
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css"
    rel="stylesheet"
  />
</svelte:head>

<div class="container my-4">
  <h2>Task Manager</h2>

  <div class="mb-3">
    <input class="form-control mb-2" bind:value={newUser.title} placeholder="Title" />
    <input class="form-control mb-2" bind:value={newUser.description} placeholder="Description" />
    <div class="form-check mb-2">
      <input class="form-check-input" type="checkbox" bind:checked={newUser.completed} />
      <label class="form-check-label">Completed</label>
    </div>
    <button class="btn btn-primary" on:click={addUser}>Add</button>
  </div>

  <table class="table table-bordered mt-4">
    <thead>
      <tr>
        <th>Title</th>
        <th>Description</th>
        <th>Completed</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      {#each users as user, index}
        <tr>
          {#if editIndex === index}
            <td><input class="form-control" bind:value={user.title} /></td>
            <td><input class="form-control" bind:value={user.description} /></td>
            <td>
              <input class="form-check-input" type="checkbox" bind:checked={user.completed} />
            </td>
            <td>
              <button class="btn btn-success btn-sm" on:click={() => saveEdit(user)}>Save</button>
              <button class="btn btn-secondary btn-sm" on:click={() => (editIndex = -1)}>Cancel</button>
            </td>
          {:else}
            <td>{user.title}</td>
            <td>{user.description}</td>
            <td>{user.completed ? "✅" : "❌"}</td>
            <td>
              <button class="btn btn-warning btn-sm" on:click={() => startEdit(index)}>Edit</button>
              <button class="btn btn-danger btn-sm" on:click={() => deleteUser(user.id)}>Delete</button>
            </td>
          {/if}
        </tr>
      {/each}
    </tbody>
  </table>
</div>