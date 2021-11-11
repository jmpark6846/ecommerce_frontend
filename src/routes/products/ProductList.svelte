<script>
import { onMount } from "svelte";
import api from "../../utils/api";
import { PaginationNav } from 'svelte-paginate'

export let navigate;
let products = [];
let productInfo = {};
let currentCategory;

onMount(async ()=> {
  await fetchProducts();
});


async function fetchProducts({page_num, category}={}){
  let params = {}
  if(page_num){
    params['page_num'] = page_num;
  }
  if(category){
    params['category'] = category
  }
  console.log(params)

  try {
    const res = await api.get(`/products/`, { params });
    products = res.data.list;
    currentCategory = category
    delete res.data['list'];
    productInfo = res.data;
  } catch (error) {
    console.error(error)    
  }
}

</script>
{#if products.length > 0 }
<section class='lg:max-w-5xl mx-auto h-full py-8'>
  <div class='flex gap-2 mb-6'>
    <button on:click={()=>fetchProducts()} class:hover:bg-gray-800={currentCategory===undefined} class:bg-gray-700={currentCategory===undefined} class:text-white={currentCategory===undefined} class='px-4 py-2 rounded-lg text-gray-700 bg-gray-100 hover:bg-gray-200'>전체</button>
    {#each productInfo.categories as category (category.id)}
      <button on:click={()=>fetchProducts({category:category.name})} class:hover:bg-gray-800={currentCategory===category.name} class:bg-gray-700={currentCategory===category.name} class:text-white={currentCategory===category.name} class='px-4 py-2 rounded-lg text-gray-700 bg-gray-100 hover:bg-gray-200'>{category.name}</button>
    {/each}
  </div>
  <div class='grid grid-cols-3 gap-8'>
    {#each products as product (product.id)}
    <div class='mb-4 hover:opacity-80 cursor-pointer' on:click="{()=>navigate(`/products/${product.id}/`)}">
      <div style='height: 280px;' >
        <img class='object-cover w-full h-full rounded bg-gray-50 border border-gray-100' src={product.image.file} alt="{product.name}" /> 
      </div>
      <div class='text-xl text-center my-2'>{product.name}</div>
    </div>
    {/each}
  </div>
  <div class="page-nav">
    <PaginationNav
      totalItems="{productInfo.count}"
      pageSize="{productInfo.page_size}"
      currentPage="{productInfo.current_page}"
      limit="{2}"
      showStepOptions="{false}"
      on:setPage="{async (e) => await fetchProducts({page_num: e.detail.page, category: currentCategory})}"
    />
  </div>
</section>
{:else}
<div></div>
{/if}