<script>
	import { onMount } from 'svelte';
	import axios from 'axios';

	let minPrice = '';
	let maxPrice = '';
	let filteredData = [];
	let currentPage = 1;
	let itemsPerPage = 5;
	let sortOrder = 'asc';

	let searchQuery = '';
	let totalPages = Math.ceil(filteredData.length / itemsPerPage);
	let mainData = [];
	let sortProperty = 'item';
	let sortDirection = 1;
	const fetchPosts = async () => {
		const response = await axios.get('https://dummyjson.com/products');
		mainData = response.data.products;
	};
	function filterData() {
		filteredData = mainData.filter((item) =>
			item.title.toLowerCase().includes(searchQuery.toLowerCase())
		);

		updatePagination();
	}
	function sortBy(property) {
		if (sortProperty === property) {
			sortDirection *= -1; // Toggle between ascending and descending
		} else {
			sortProperty = property;
			sortDirection = 1; // Reset to ascending when changing the sorting property
		}

		filteredData = filteredData.slice().sort((a, b) => {
			return sortDirection * (a[property] > b[property] ? 1 : -1);
		});
		updatePagination();
	}
	function sortData() {
		if (sortOrder === 'asc') {
			filteredData.sort((a, b) => console.log(a, b, 'a,b'));
		} else {
			filteredData.sort((a, b) => (a.name < b.name ? 1 : -1));
		}
		filteredData = filteredData.slice().sort((a, b) => {
			return sortDirection * (a[name] > b[name] ? 1 : -1);
		});
	}
	function updatePagination() {
		totalPages = Math.ceil(filteredData.length / itemsPerPage);
		currentPage = Math.min(currentPage, totalPages);
	}

	function changeSortOrder() {
		sortOrder = sortOrder === 'asc' ? 'desc' : 'asc';

		sortBy('name');
	}

	function goToPage(pageNumber) {
		currentPage = pageNumber;
	}
	function visibleData() {
		const startIndex = (currentPage - 1) * itemsPerPage;
		const endIndex = startIndex + itemsPerPage;
		return filteredData.slice(startIndex, endIndex);
	}
	onMount(() => {
		fetchPosts().then((res) => {
			// After fetching data, filter and sort
			filterData();
			sortData();
		});
	});

	// const byprice = () => {
	// 	filteredData = mainData.filter((item) => item.price > minPrice && item.price < maxPrice);
	// };

	function validatePrices() {
		if (minPrice !== '' && maxPrice !== '') {
			if (parseInt(minPrice) > parseInt(maxPrice)) {
				alert('Min price cannot be greater than max price.');
				return false;
			}
		}
		return true;
	}

	async function byprice() {
		if (validatePrices()) {
			filteredData = mainData.filter((item) => item.price > minPrice && item.price < maxPrice);
		}
	}
</script>

<div class="container mx-auto px-4 sm:px-6 lg:px-8">
	<div class="flex flex-col space-y-4 sm:flex-row sm:space-y-0 sm:space-x-4">
		<div class="flex items-center">
			<label for="minPrice" class="mr-2">Min Price:</label>
			<input type="number" id="minPrice" bind:value={minPrice} class="border rounded px-2 py-1" />
		</div>

		<div class="flex items-center">
			<label for="maxPrice" class="mr-2">Max Price:</label>
			<input type="number" id="maxPrice" bind:value={maxPrice} class="border rounded px-2 py-1" />
		</div>
		<div>
			<button on:click={byprice} class="mt-1 px-2 py-1 bg-blue-500 text-white rounded"
				>SUBMIT</button
			>
		</div>
	</div>

	<br />

	<input type="text" placeholder="Search" bind:value={searchQuery} on:input={filterData} />
	<br />
	<br />
	<table class="min-w-full divide-y divide-gray-200">
		<thead class="bg-gray-50">
			<tr>
				<th
					scope="col"
					class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
					on:click={changeSortOrder}
				>
					title
					{#if sortOrder === 'asc'}
						<span class="ml-1">&#8593;</span>
					{:else}
						<span class="ml-1">&#8595;</span>
					{/if}
				</th>
				<th
					scope="col"
					class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
				>
					brand
				</th>
				<th
					scope="col"
					class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
				>
					image
				</th>
				<th
					scope="col"
					class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
				>
					price
				</th>
				<th
					scope="col"
					class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
				>
					stock
				</th>
			</tr>
		</thead>
		<tbody class="bg-white divide-y divide-gray-200">
			{#each filteredData as item (item.id)}
				<tr key={item.id}>
					<td class="px-6 py-4 whitespace-nowrap">
						<div class="flex items-center">
							<div class="ml-4">
								<div class="text-sm font-medium text-gray-900">{item.title}</div>
							</div>
						</div>
					</td>
					<td class="px-6 py-4 whitespace-nowrap">
						<div class="text-sm text-gray-500">{item.brand}</div>
					</td>
					<td class="px-6 py-4 whitespace-nowrap">
						<div class="text-sm text-gray-500 w-100">
							<img src={item.thumbnail} alt="image" />
						</div>
					</td>
					<td class="px-6 py-4 whitespace-nowrap">
						<div class="text-sm text-gray-500">{item.price}</div>
					</td>
					<td class="px-6 py-4 whitespace-nowrap">
						<div class="text-sm text-gray-500">{item.stock}</div>
					</td>
				</tr>
			{/each}
		</tbody>
	</table>

	<div class="mt-4">
		<nav class="flex items-center justify-between">
			<div class="flex-1 flex justify-between sm:hidden">
				<button
					onclick={() => goToPage(currentPage - 1)}
					disabled={currentPage === 1}
					class="relative inline-flex items-center px-4 py-2 border border-gray-300 text-sm font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50"
				>
					Previous
				</button>
				<button
					onclick={() => goToPage(currentPage + 1)}
					disabled={currentPage === totalPages}
					class="ml-3 relative inline-flex items-center px-4 py-2 border border-gray-300 text-sm font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50"
				>
					Next
				</button>
			</div>
			<div class="hidden sm:flex-1 sm:flex sm:items-center sm:justify-between">
				<div>
					<p class="text-sm text-gray-700">
						Showing
						<span class="font-medium">{(currentPage - 1) * itemsPerPage + 1}</span>
						to
						<span class="font-medium"
							>{(currentPage - 1) * itemsPerPage + visibleData().length}</span
						>
						of
						<span class="font-medium">{filteredData.length}</span>
						results
					</p>
				</div>
				<div>
					<nav
						class="relative z-0 inline-flex rounded-md shadow-sm -space-x-px"
						aria-label="Pagination"
					>
						<button
							onclick={() => goToPage(currentPage - 1)}
							disabled={currentPage === 1}
							class="relative inline-flex items-center px-2 py-2 rounded-l-md border border-gray-300 bg-white text-sm font-medium text-gray-500 hover:bg-gray-50"
						>
							<span class="sr-only">Previous</span>
							<svg
								class="h-5 w-5"
								xmlns="http://www.w3.org/2000/svg"
								viewBox="0 0 20 20"
								fill="currentColor"
								aria-hidden="true"
							>
								<path
									fill-rule="evenodd"
									d="M10.707 3.293a1 1 0 010 1.414L6.414 9H13a8 8 0 110 16H7a1 1 0 010-2h6a6 6 0 100-12H6.414l4.293 4.293a1 1 0 01-1.414 1.414l-6-6a1 1 0 010-1.414l6-6a1 1 0 011.414 0z"
									clip-rule="evenodd"
								/>
							</svg>
						</button>
						{#each Array.from({ length: totalPages }, (_, i) => i + 1) as page}
							<button
								onclick={() => goToPage(page)}
								class="{currentPage === page
									? 'bg-blue-500 text-white'
									: 'bg-white text-gray-700 hover:bg-gray-50'} relative inline-flex items-center px-4 py-2 border border-gray-300 text-sm font-medium"
							>
								{page}
							</button>
						{/each}
						<button
							onclick={() => goToPage(currentPage + 1)}
							disabled={currentPage === totalPages}
							class="relative inline-flex items-center px-2 py-2 rounded-r-md border border-gray-300 bg-white text-sm font-medium text-gray-500 hover:bg-gray-50"
						>
							<span class="sr-only">Next</span>
							<svg
								class="h-5 w-5"
								xmlns="http://www.w3.org/2000/svg"
								viewBox="0 0 20 20"
								fill="currentColor"
								aria-hidden="true"
							>
								<path
									fill-rule="evenodd"
									d="M9.293 16.707a1 1 0 010-1.414L13.586 11H7a8 8 0 110-16h6a1 1 0 010 2H7a6 6 0 100 12h6.586l-4.293-4.293a1 1 0 011.414-1.414l6 6a1 1 0 010 1.414l-6 6a1 1 0 01-1.414 0z"
									clip-rule="evenodd"
								/>
							</svg>
						</button>
					</nav>
				</div>
			</div>
		</nav>
	</div>
</div>

<style>
</style>
