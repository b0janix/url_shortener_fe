<template>    
	<form @submit.prevent="submitForm">
	<div>
        <h1>
            Shorten your urls
        </h1>
    </div>
        <div class="form-control">
			<label for="url">Your URL</label>
			<input
				id="url"
				name="url"
				type="text"
				v-model.trim="urlField"
				autocomplete="off"
			/>
		</div>
		<div>
			<button>Add URL</button>
			<p id="success" class="invisible">{{ success }}</p>
			<p id="failure" class="invisible"> {{ failure }}</p>
		</div>
	</form>

    <div class="display-urls">
		<div style="display: flex; flex-direction: column; align-items: center;">
			<button class="list-button" v-for="url in urls" :key="url.id" @click="openUrl(url.full_long_url)"> 
				{{ url.full_short_url }} 
			</button>
		</div>
		<div class="pagination-buttons-container">
			<button v-if="prev" @click="getData(prev)">
				Prev
			</button>
			<button v-if="next" @click="getData(next)">
				Next
			</button>
		</div>
    </div>
</template>

<script>
export default {
	data() {
		return {
			urlField: '',
            urls: [],
			prev: '',
			next: '',
			success: '',
			failure: ''
		};
	},
    mounted() {
        this.getData(process.env.VUE_APP_BE_HOST)
    },
	methods: {
		async submitForm() {
			try {
				let response = await fetch(
				`${process.env.VUE_APP_BE_HOST}/store`,
					{
						method: "POST",
						mode: "cors",
						headers: {
							"Content-Type": "application/json",
							"Accept": "application/json",
							"X-PINGOTHER": "pingpong",
						},
						body: JSON.stringify({url: this.urlField})
					}
				);
				response.json()
				.then((r) => {
					this.urlField = '' 
					this.success = r.message
					this.failure = ''
					this.showMessage()
				})
				.catch((e) =>  {
					this.failure = e
					this.success = ''
					this.showMessage()
				})
			} catch(e) {
				this.failure = e
				this.success = ''				
			}
			this.showMessage()			
		},
		showMessage() {
			document.querySelector('#success').style.display = 'none';
			document.querySelector('#failure').style.display = 'none';

			this.getData(process.env.VUE_APP_BE_HOST)

			if (this.success.length > 0) {
				document.querySelector('#success').style.display = 'block';
				document.querySelector('#failure').style.display = 'none';
			}
			
			if (this.failure.length > 0) {
				document.querySelector('#success').style.display = 'none';
				document.querySelector('#failure').style.display = 'block';
			}
			
			setTimeout(() => {
				document.querySelector('#success').style.display = 'none';
				document.querySelector('#failure').style.display = 'none';
			}, 3000)
			
		},		
		changePage(url) {
			this.getData(url)
		},
		async getData(url) {
			try {
				let response = await fetch(url);				
				response.json().then((r) => {					
				this.prev = r.links.prev
				this.next = r.links.next
				this.urls = r.data
			})
			} catch(e) {
				console.log(e)
			}
			
		},
		openUrl(url) {
			window.location.href = url
		}        
	},
};
</script>

<style scoped>
*:focus {
	outline: none;
}
form, .display-urls {
	margin: 2rem auto;
	max-width: 40rem;
	border-radius: 12px;
	box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
	padding: 2rem;
	background-color: #d6d6d6;
}

.form-control {
	margin: 0.5rem 0;
}

.form-control.invalid input {
	border-color: #b6524b;
}

.form-control.invalid label {
	color: #b6524b;
}

label {
	font-weight: bold;
}

h2 {
	font-size: 1rem;
	margin: 0.5rem 0;
}

input,
select {
	display: block;
	width: 100%;
	font: inherit;
	margin-top: 0.5rem;
	background: rgba(255, 255, 255, 0.123);
	border: 2px solid gray;
	padding: 0.5em;
}

select {
	width: auto;
}

button {
	font: inherit;
	border: 1px solid #0076bb;
	background-color: #0076bb;
	color: white;
	cursor: pointer;
	padding: 0.75rem 2rem;
	border-radius: 30px;
}

.list-button {
    margin: 0.75rem 2rem;
	background-color: #b6524b; 
}

button:hover,
button:active {
	border-color: #002350;
	background-color: #002350;
}

.pagination-buttons-container {
	display: flex; 
	justify-content: space-between;
	margin-top: 20px;
}

.invisible {
	display: none;
}
</style>