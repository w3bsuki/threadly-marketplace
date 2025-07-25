<script lang="ts">
	import { X, Filter, Sparkles } from 'lucide-svelte';
	import { goto } from '$app/navigation';
	import { cn } from '$lib/utils';
	
	let selectedFilters = $state({
		price: '',
		size: '',
		brand: '',
		condition: '',
		sort: 'recent'
	});
	
	let showAllFilters = $state(false);
	
	const quickFilters = [
		{
			type: 'price',
			icon: '💷',
			options: [
				{ label: 'Under £20', value: '0-20' },
				{ label: '£20-50', value: '20-50' },
				{ label: '£50-100', value: '50-100' },
				{ label: '£100+', value: '100-999' }
			]
		},
		{
			type: 'size',
			icon: '📏',
			options: [
				{ label: 'XS', value: 'xs' },
				{ label: 'S', value: 's' },
				{ label: 'M', value: 'm' },
				{ label: 'L', value: 'l' },
				{ label: 'XL', value: 'xl' }
			]
		},
		{
			type: 'brand',
			icon: '🏷️',
			options: [
				{ label: 'Nike', value: 'nike' },
				{ label: 'Adidas', value: 'adidas' },
				{ label: 'Zara', value: 'zara' },
				{ label: 'H&M', value: 'hm' },
				{ label: 'Gucci', value: 'gucci' }
			]
		},
		{
			type: 'condition',
			icon: '✨',
			options: [
				{ label: 'New', value: 'new' },
				{ label: 'Like New', value: 'likenew' },
				{ label: 'Good', value: 'good' },
				{ label: 'Fair', value: 'fair' }
			]
		}
	];
	
	const sortOptions = [
		{ label: 'Most Recent', value: 'recent', icon: '🆕' },
		{ label: 'Price: Low to High', value: 'price-low', icon: '📈' },
		{ label: 'Price: High to Low', value: 'price-high', icon: '📉' },
		{ label: 'Most Popular', value: 'popular', icon: '🔥' }
	];
	
	function applyFilter(type: string, value: string) {
		selectedFilters[type] = selectedFilters[type] === value ? '' : value;
		updateUrl();
	}
	
	function clearAllFilters() {
		selectedFilters = {
			price: '',
			size: '',
			brand: '',
			condition: '',
			sort: 'recent'
		};
		updateUrl();
	}
	
	function updateUrl() {
		const params = new URLSearchParams();
		
		Object.entries(selectedFilters).forEach(([key, value]) => {
			if (value && value !== 'recent') {
				params.set(key, value);
			}
		});
		
		goto(`/browse${params.toString() ? '?' + params.toString() : ''}`);
	}
	
	const activeFilterCount = $derived(
		Object.values(selectedFilters).filter(v => v && v !== 'recent').length
	);
</script>

<section class="bg-gray-50 border-b sticky top-[64px] md:top-[80px] z-30">
	<div class="container px-4 py-2">
		<!-- Mobile Layout -->
		<div class="md:hidden">
			<div class="flex items-center gap-2 overflow-x-auto scrollbar-hide">
				<!-- Filter Toggle Button -->
				<button
					onclick={() => showAllFilters = !showAllFilters}
					class={cn(
						"flex items-center gap-1.5 px-3 py-1.5 rounded-full border whitespace-nowrap transition-all duration-200 flex-shrink-0 text-xs",
						activeFilterCount > 0
							? "bg-orange-500 text-white border-orange-500 hover:bg-orange-600"
							: "bg-white border-gray-200 hover:border-orange-300 hover:bg-orange-50"
					)}
				>
					<Filter class="h-3.5 w-3.5" />
					<span class="font-medium">Filters</span>
					{#if activeFilterCount > 0}
						<span class="bg-white text-orange-500 rounded-full px-1.5 py-0.5 text-[10px] font-bold">
							{activeFilterCount}
						</span>
					{/if}
				</button>
				
				<!-- Quick Filter Pills -->
				{#each quickFilters as filter}
					<div class="relative">
						<select
							value={selectedFilters[filter.type]}
							onchange={(e) => applyFilter(filter.type, e.currentTarget.value)}
							class={cn(
								"appearance-none pl-6 pr-6 py-1.5 rounded-full text-xs font-medium border whitespace-nowrap cursor-pointer transition-all duration-200",
								selectedFilters[filter.type]
									? "bg-orange-100 border-orange-300 text-orange-700"
									: "bg-white border-gray-200 hover:border-orange-300 hover:bg-orange-50"
							)}
						>
							<option value="">{filter.type.charAt(0).toUpperCase() + filter.type.slice(1)}</option>
							{#each filter.options as option}
								<option value={option.value}>{option.label}</option>
							{/each}
						</select>
						<span class="absolute left-2 top-1/2 -translate-y-1/2 text-sm pointer-events-none">
							{filter.icon}
						</span>
						{#if selectedFilters[filter.type]}
							<button
								onclick={() => applyFilter(filter.type, '')}
								class="absolute right-1.5 top-1/2 -translate-y-1/2"
							>
								<X class="h-2.5 w-2.5 text-orange-600" />
							</button>
						{/if}
					</div>
				{/each}
				
				<!-- Sort -->
				<select
					value={selectedFilters.sort}
					onchange={(e) => { selectedFilters.sort = e.currentTarget.value; updateUrl(); }}
					class="appearance-none pl-6 pr-3 py-1.5 rounded-full text-xs font-medium border bg-white border-gray-200 hover:border-orange-300 hover:bg-orange-50 whitespace-nowrap"
				>
					{#each sortOptions as option}
						<option value={option.value}>{option.label}</option>
					{/each}
				</select>
			</div>
		</div>
		
		<!-- Desktop Layout -->
		<div class="hidden md:flex items-center justify-between">
			<div class="flex items-center gap-3">
				{#each quickFilters as filter}
					<div class="relative group">
						<button
							onclick={() => showAllFilters = !showAllFilters}
							class={cn(
								"flex items-center gap-2 px-4 py-2 rounded-lg border font-medium transition-all duration-200",
								selectedFilters[filter.type]
									? "bg-orange-100 border-orange-300 text-orange-700 hover:bg-orange-200"
									: "bg-white border-gray-200 hover:border-orange-300 hover:bg-orange-50"
							)}
						>
							<span>{filter.icon}</span>
							<span>{filter.type.charAt(0).toUpperCase() + filter.type.slice(1)}</span>
							{#if selectedFilters[filter.type]}
								<span class="text-sm">: {filter.options.find(o => o.value === selectedFilters[filter.type])?.label}</span>
								<X class="h-3 w-3 ml-1" onclick={(e) => { e.stopPropagation(); applyFilter(filter.type, ''); }} />
							{/if}
						</button>
						
						<!-- Dropdown -->
						<div class="absolute top-full left-0 mt-2 bg-white rounded-lg shadow-lg border border-gray-200 p-2 min-w-[150px] opacity-0 invisible group-hover:opacity-100 group-hover:visible transition-all duration-200 z-10">
							{#each filter.options as option}
								<button
									onclick={() => applyFilter(filter.type, option.value)}
									class={cn(
										"w-full text-left px-3 py-2 rounded-md text-sm transition-colors duration-150",
										selectedFilters[filter.type] === option.value
											? "bg-orange-100 text-orange-700"
											: "hover:bg-gray-100"
									)}
								>
									{option.label}
								</button>
							{/each}
						</div>
					</div>
				{/each}
				
				{#if activeFilterCount > 0}
					<button
						onclick={clearAllFilters}
						class="text-sm text-gray-600 hover:text-orange-600 font-medium ml-2"
					>
						Clear all
					</button>
				{/if}
			</div>
			
			<!-- Sort Dropdown -->
			<div class="flex items-center gap-2">
				<Sparkles class="h-4 w-4 text-gray-400" />
				<select
					value={selectedFilters.sort}
					onchange={(e) => { selectedFilters.sort = e.currentTarget.value; updateUrl(); }}
					class="bg-white border border-gray-200 rounded-lg px-3 py-2 text-sm font-medium hover:border-orange-300 focus:outline-none focus:ring-2 focus:ring-orange-500/20"
				>
					{#each sortOptions as option}
						<option value={option.value}>{option.icon} {option.label}</option>
					{/each}
				</select>
			</div>
		</div>
	</div>
	
	<!-- Expanded Filters (Mobile) -->
	{#if showAllFilters}
		<div class="md:hidden border-t bg-white px-4 py-4">
			<div class="space-y-4">
				{#each quickFilters as filter}
					<div>
						<h3 class="text-sm font-semibold text-gray-700 mb-2 flex items-center gap-1">
							<span>{filter.icon}</span>
							{filter.type.charAt(0).toUpperCase() + filter.type.slice(1)}
						</h3>
						<div class="flex flex-wrap gap-2">
							{#each filter.options as option}
								<button
									onclick={() => applyFilter(filter.type, option.value)}
									class={cn(
										"px-3 py-1.5 rounded-full text-sm font-medium border transition-all duration-200",
										selectedFilters[filter.type] === option.value
											? "bg-orange-500 text-white border-orange-500"
											: "bg-white border-gray-200 hover:border-orange-300 hover:bg-orange-50"
									)}
								>
									{option.label}
								</button>
							{/each}
						</div>
					</div>
				{/each}
			</div>
		</div>
	{/if}
</section>

<style>
	.scrollbar-hide {
		-ms-overflow-style: none;
		scrollbar-width: none;
	}
	.scrollbar-hide::-webkit-scrollbar {
		display: none;
	}
	
	/* Custom select arrow */
	select {
		background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%23666' d='M10.293 3.293L6 7.586 1.707 3.293A1 1 0 00.293 4.707l5 5a1 1 0 001.414 0l5-5a1 1 0 10-1.414-1.414z'/%3E%3C/svg%3E");
		background-repeat: no-repeat;
		background-position: right 0.5rem center;
		background-size: 1rem;
	}
</style>