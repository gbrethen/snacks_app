<script>
    import Fa from 'svelte-fa';
    import { faTrash } from '@fortawesome/free-solid-svg-icons';
    import { faEdit } from '@fortawesome/free-solid-svg-icons';
    import { onMount } from 'svelte';
    export let data;
    export let deleteFunction = () => {};
    export let resetSnacks = () => {};

    let update_url = `http://localhost:4000/snacks/`;
    let normal;
    let update;

    onMount(() => {
        normal.style.display = "block";
        update.style.display = "none";
    });

    function showUpdate(e) {
        normal.style.display = "none";
        update.style.display = "block";
    }

    function cancelUpdate() {
        normal.style.display = "block";
        update.style.display = "none";
    }

    async function updateSnack(e) {
		//alert("id: " + e.target.id + ", brand: " + e.target.brand.value);
        const data = { "snack": {
                "brand": e.target.brand.value,
                "style": e.target.style.value,
                "origin": e.target.origin.value,
                "quantity": e.target.quantity.value
            }
        }

		const putData = JSON.stringify(data);
		//console.log(putData);

		//send to api
		const res = await fetch(update_url + e.target.id, {
			method: 'PUT',
			mode: 'cors',
			headers: { 'Content-Type': 'application/json'},
			referrerPolicy: 'no-referrer',
			body: putData
		})

		const json = await res.json();
		//const result = JSON.stringify(json);

		//console.log("res: " + result);

		resetSnacks();
	}
    
</script>

<form id={data.id} on:submit|preventDefault={updateSnack}>
    <div class="card m-2" style="max-width: 16rem;">
        <div class="card-header">
            <div class="row">
                <h5 class="col-8">Brand: {data.brand}</h5>
                <div class="col-4 text-end">
                    <a href="#" on:click={showUpdate} style="color: #000;"><Fa icon={faEdit} /></a>&nbsp;
                    <a href="#" on:click={deleteFunction(data.id)} style="color: #000;"><Fa icon={faTrash} /></a>
                </div>
            </div>
        </div>
        <div class="card-body" bind:this={normal}>
            <p class="card-text">
                Style: {data.style}<br>
                Origin: {data.origin}<br>
                Quantity: {data.quantity}
            </p>
        </div>
        <div class="card-body" bind:this={update}>
            <div class="card-text">
                <div class="row g-2 align-items-center">
                    <label for="style" class="col-sm-4 col-form-label">Style:</label>
                    <div class="col-sm-8">
                        <input type="text" class="form-control form-control-sm" name="style" id="style" value="{data.style}"/>
                    </div>
                    <div class="d-none">
                        <input type="text" class="form-control form-control-sm" name="brand" id="brand" value="{data.brand}"/>
                    </div>
                </div>
                <div class="row g-2 align-items-center">
                    <label for="origin" class="col-sm-4 col-form-label">Origin:</label>
                    <div class="col-sm-8">
                        <input type="text" class="form-control form-control-sm" name="origin" id="origin" value="{data.origin}"/>
                    </div>
                </div>
                <div class="row g-2 align-items-center">
                    <label for="quantity" class="col-sm-4 col-form-label">Quantity:</label>
                    <div class="col-sm-3">
                        <input type="text" class="form-control form-control-sm" name="quantity" id="quantity" value="{data.quantity}"/>
                    </div>
                </div>
                <div class="row g-2 mt-2 text-end">
					<div class="col-sm-12">
                        <button type="button" class="btn btn-danger" on:click={cancelUpdate}>Cancel</button>
						<button type="submit" class="btn btn-primary">Update</button>
					</div>
				</div>
            </div>
        </div>
    </div>
</form>
