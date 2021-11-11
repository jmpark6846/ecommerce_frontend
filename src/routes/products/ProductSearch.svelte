<script>
import { onMount } from "svelte";
import api from "../../utils/api";
import { PaginationNav } from 'svelte-paginate'

export let search_term;
export let navigate;

let products = [];
let productInfo = {};

$: {
  if(search_term){
    fetchSearchResult();
  }
}

onMount(async ()=> {
  await fetchSearchResult();
});


async function fetchSearchResult(page_num=1){
  try {
    const res = await api.get(`/products/`, {params: {search: search_term, page_num}});
    products = res.data.list;
    delete res.data['list'];
    productInfo = res.data;
  } catch (error) {
    console.error(error)    
  }
}

</script>
<section class='lg:max-w-5xl mx-auto h-full py-8'>
  <h1 class='text-3xl font-bold mb-4'>검색 결과: {search_term}</h1>

  {#if productInfo.count < 1}
    <div>검색어에 해당하는 상품이 없습니다.</div>
  {:else}
  <div class='grid grid-cols-3 gap-8'>
    {#each products as product (product.id)}
    <div class='mb-4 hover:opacity-80 cursor-pointer' on:click="{()=>navigate(`/products/${product.id}/`)}">
      <div style='height: 280px;' >
        <img class='object-cover w-full h-full rounded bg-gray-50 border border-gray-100' src={product.image.file} alt="{product.name}" /> 
      </div>
      <div class='text-2xl text-center my-2'>{product.name}</div>
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
      on:setPage="{async (e) => await fetchSearchResult(e.detail.page)}"
    />
  </div>
  {/if}
  
</section>