<script>
	import Snack from './snack.svelte';
	import Flex from 'svelte-flex';
	import { toasts, ToastContainer, FlatToast, BootstrapToast } from 'svelte-toasts';
	
	let url = `http://localhost:4000/snacks`;
	let search_url = `http://localhost:4000/snacks?brand=`;
	let single_url = `http://localhost:4000/snacks/`;
	let result;
	let mySnacks = [];
	let cols = 3;

	async function getSnacks() {
		let response = await fetch(url);
		let json = await response.json();
		let data = json['data'];
		return data;
	}

	async function searchSnacks(val) {
		let response = await fetch(search_url+val);
		let json = await response.json();
		let data = json['data'];
		if (Object.keys(data).length > 0) {
			mySnacks = [];
		}
		return data;
	}

	function buttonHandler(e) {
		mySnacks = [];
		result = getSnacks();
		
		var frm = document.getElementById("divAddSnack");
		var rslt = document.getElementById("divSnacks");
		if (rslt.style.display === "none") {
			rslt.style.display = "block";
			frm.style.display = "none";
		}
	}

	function showSnackForm() {
		var frm = document.getElementById("divAddSnack");
		var rslt = document.getElementById("divSnacks");
		if (frm.style.display === "none") {
			rslt.style.display = "none";
			frm.style.display = "block";
		}
	}

	function searchByBrand(){
		if (txt_brand.value != "") {
			result = searchSnacks(txt_brand.value);
			if (Object.keys(result).length === 0) {
				toasts.error("Snacks", "No snacks were found with that brand!");
			} 
			txt_brand.value = "";
		}else{
			//use toast to display message
			toasts.error("Error", "Please enter a brand to search!");
		}
	}

	async function submitSnack(e) {
		const formData = new FormData(e.target);
		const data = {};

		for (let field of formData) {
			const [key, value] = field;
			data[key] = value;
		}

		const postData = JSON.stringify({ "snack":data });
		console.log(postData);

		//send to api
		const res = await fetch(url, {
			method: 'POST',
			mode: 'cors',
			headers: { 'Content-Type': 'application/json'},
			referrerPolicy: 'no-referrer',
			body: postData
		})

		const json = await res.json();
		result = JSON.stringify(json);

		console.log("res: " + result);

		e.target.reset();
	}

	async function deleteSnack(id) {
		//send to api
		console.log("id: " + id);
		const res = await fetch(single_url + id, {
			method: 'DELETE',
			mode: 'cors'
		})

		console.log("res: " + res);

		buttonHandler();
	}

</script>
<nav id="defaultNav" name="defaultNav" class="navbar navbar-expand-lg navbar-dark bg-dark  nav-pills">
	<div class="container-fluid text-start">
		<a class="navbar-brand" href="#">
			<img src="/images/logo.png" alt="Store Logo"/>
		</a>
		<div class="collapse navbar-collapse" id="navbarSupportedContent">
			<ul class="navbar-nav me-auto mb-2">
				<li class="nav-item">
					<a class="nav-link" href="#" on:click={buttonHandler}>Snacks</a>
				</li>
				<li class="nav-item">
					<a class="nav-link" href="#" on:click={showSnackForm}>Add Snack</a>
				</li>
			</ul>
		</div>
		<form class="d-flex">
			<input class="form-control me-2" type="search" id="txt_brand" placeholder="Search by Brand" aria-label="Search">
			<button class="btn btn-primary" type="submit" on:click="{searchByBrand}">Search</button>
		</form>
	</div>
</nav>
<div id="divSnacks">
	{#if result === undefined}
		<p></p>
	{:else}
		{#await result}
			<p>Loading...</p>
		{:then snacks}
			{#each snacks as snack}
				{(() => {
					mySnacks.push(snack)
					return ''; //so svelte does not print.
				})()}
			{/each}

			{#each mySnacks as theSnack, i}
				{#if i % cols === 0}
					<div class="row row-cols-5 mt-3">
						{#each Array(cols) as _,j}
							<div class="col">
								{#if mySnacks[i/cols*cols + j]}
									<Snack data={mySnacks[i/cols*cols + j]} deleteFunction={() => deleteSnack(mySnacks[i/cols*cols + j].id)} resetSnacks={() => buttonHandler()} />
								{/if}
							</div>
						{/each}
					</div>
				{/if}
			{/each}
		{/await}
	{/if}
</div>
<div id="divAddSnack" style="display: none; width: 45%; margin-left: 5px; margin-top: 15px;">
	<div class="card" style="width: 32rem;">
		<div class="card-header text-center">Add New Snack</div>
		<div class="card-body">
			<form id="frmSnack" on:submit|preventDefault={submitSnack}>
				<div class="row g-3 mb-3 align-items-center">
					<label for="brand" class="col-sm-2 col-form-label">Brand:</label>
					<div class="col-sm-10">
						<input type="text" class="form-control" name="brand" id="brand" />
					</div>
				</div>
				<div class="row g-3 mb-3 align-items-center">
					<label for="style" class="col-sm-2 col-form-label">Style:</label>
					<div class="col-sm-10">
						<input type="text" class="form-control" name="style" id="style" />
					</div>
				</div>
				<div class="row g-3 mb-3 align-items-center">
					<label for="origin" class="col-sm-2 col-form-label">Origin:</label>
					<div class="col-sm-10">
						<input type="text" class="form-control" name="origin" id="origin" />
					</div>
				</div>
				<div class="row g-3 mb-3 align-items-center">
					<label for="quantity" class="col-sm-2 col-form-label">Quantity: </label>
					<div class="col-sm-10">
						<input type="text" class="form-control" name="quantity" id="quantity" />
					</div>
				</div>
				<div class="row g-3 mb-3">
					<div class="col-md-6">
						<button type="submit" class="btn btn-primary">Submit</button>
					</div>
				</div>
			</form>
		</div>
	</div>
</div>
<ToastContainer placement="bottom-right" let:data={data}>
	<FlatToast {data} />
</ToastContainer>
<style>
	@media (min-width: 640px) {
		
	}

	#defaultNav {
		-webkit-box-shadow: 0px 5px 17px 0px rgba(89,84,89,1);
		-moz-box-shadow: 0px 5px 17px 0px rgba(89,84,89,1);
		box-shadow: 0px 5px 17px 0px rgba(89,84,89,1);
	}
</style>