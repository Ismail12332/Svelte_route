<script>
  import { Router, Route, Link, } from "svelte-navigator";
	import Glav from "./views/Glav.svelte";
	import Login from "./Login.svelte";
	import { onMount } from "svelte";
  import EditProject from "./EditProject.svelte";

	let showModal = false;
	let isLoggedIn = localStorage.getItem('user_id'); // Check if the user is logged in
    

    


  //// Function to handle user logout
  //const logout = () => {
  //  localStorage.removeItem('user_id');
  //  isLoggedIn = false;
  //};
//
  //// Event handler for successful login
  //const handleLoginSuccess = (event) => {
  //  isLoggedIn = true;
  //};

    
    
</script>


<Router>

        <Route path='/'>
            <Login />
        </Route>

        <Route path='/glav'>
            <Glav />
        </Route>

        <Route path='/EditProject/:project_id'>
            <EditProject/>
        </Route>




</Router>

<!--
    <body>
        
        <main>
            {#if isLoggedIn}
                <div>
                    <Glav />
                    <button on:click={logout}>Logout</button>
                </div>
            {:else}
                <div class="for-login">
                    <Login on:loginSuccess={handleLoginSuccess} />
                    <button class="for-fxod" on:click={() => (showModal = true)}>Register</button>
                </div>
            {/if}
            {#if showModal}
                <Register on:close={() => (showModal = false)} />
            {/if}
        </main>
        
    </body>
-->

<style>
	

</style>
