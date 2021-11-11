<script>
import { onMount, createEventDispatcher } from "svelte";
import { useParams } from "svelte-navigator";
import Select from 'svelte-select'
import api from '../../utils/api'
import NumberSelector from '../../components/NumberSelector.svelte'
import { user } from '../../store'
export let navigate;

let optionSelector;
const params = useParams();
let loading = true;
let product = {};
let _selected_option = null;
let selectedOptions = [];
let totalProductAmount = 0;
let totalProductQty = 0;

onMount(async ()=> {
  try {
    const res = await api.get('/products/'+$params.product_id);
    product = res.data;
    loading = false;
  } catch (error) {
    console.error(error)    
  }
})

function labelForOption(option){
  const price_diff = -1 * (product.price - option.price)
  if(price_diff == 0){
    return option.value
  }
  return `${option.value} (${price_diff > 0 ? '+' : ''}${price_diff}원)`

}

function getProductOptionsForSelect(options){
  return options.map(option => ({ value: option, label: labelForOption(option) }))
}

function handleSelectOption(e){
  const selectedOption = e.detail.value
  for(const o of selectedOptions){
    if (selectedOption.id === o.id){
      alert('이미 선택된 옵션입니다.')
      return;
    }
  }

  selectedOptions = [ ...selectedOptions, { ...selectedOption, qty: 1 } ]
  optionSelector.handleClear();
}

function handleDeleteOption(option_id){
  selectedOptions = selectedOptions.filter(option => option.id != option_id)
}

$:{
  totalProductAmount = 0
  totalProductQty = 0

  for(const option of selectedOptions){
    totalProductAmount += option.price * option.qty;
    totalProductQty += option.qty;
  }
  totalProductAmount += product.shipping_fee;
}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     

async function handleClickAddToCart(options){
  if(selectedOptions.length < 1){
    alert('상품 옵션을 선택하지 않았습니다. 먼저 상품 옵션을 선택해주세요.');
    return;
  }
  
  if($user){
    let data = []
    
    for(const option of options){
      data.push({
        cart: $user.shoppingcart.id,
        option: option.id,
        qty: option.qty
      })
    }

    try {
      const res = await api.post(`/accounts/${$user.pk}/cart/`, data);
      console.log(res.data)
      if(confirm('장바구니에 추가하였습니다. 장바구니로 이동하시겠습니까?')){
        navigate('/cart/', { replace: true })
      }
    } catch (error) {
      console.error(error)
    }
  }else{

  }
  
}

</script>
<svelte:head>
  Product Detail{loading && " - "+product.name}
</svelte:head>
{#if loading }
  <div>loading...</div>
{:else}
<section class='lg:max-w-5xl mx-auto h-full py-8'>
  <div class='product-info grid grid-cols-1 lg:grid-cols-2 space-y-4'>
    <div class='product-image-box col-start-1 flex justify-center'>
      <img class='object-cover max-w-lg max-h-72 rounded bg-gray-50 border border-gray-100' src={product.productimage_set[0].file} alt="artwork" /> 
    </div>
    <div class='product-info-box col-start-1 row-start-2 lg:col-start-2 lg:row-start-1 px-4 '>
      <h1 class='text-2xl font-bold'>{product.name}</h1>
      <div class='mb-4'>
        <h3 class='text-2xl font-extrabold text-right '>{product.price}원</h3>
        <div class='text-right text-gray-700'>택배배송비 {product.shipping_fee}원</div>
      </div>
      <Select bind:this={optionSelector} isSearchable={false} placeholder='상품 옵션을 선택하세요.' items={getProductOptionsForSelect(product.productoption_set)} on:select={handleSelectOption}/>
      <div class="mb-4">
        {#each selectedOptions as option, index (index)}
        <div class='mt-4 border border-gray-300 p-4 rounded '>
          <div class='mb-2'>{option.value}</div>
          <div class='flex justify-between items-center'>
            <NumberSelector bind:num="{option.qty}"/>
            <div class='flex space-x-4 items-center'>
              <div>
                {option.price * option.qty}원
              </div>
              <button on:click="{()=>handleDeleteOption(option.id)}">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </button>
            </div>
          </div>
        </div>
        {/each}
      </div>

      {#if totalProductQty}
      <div class='flex justify-between mb-4'>
        <div class='font-semibold'>총 상품 금액</div>
        <div class='flex'>
          <div class='text-gray-500 mr-2'>총 상품 갯수 {totalProductQty}개</div>
          <div class='font-semibold'>{totalProductAmount}원</div>
        </div>
      </div>
      {/if}

      <button class='flex items-center space-x-2 rounded px-4 py-2 bg-green-500 hover:bg-green-600 text-white' on:click={async ()=>handleClickAddToCart(selectedOptions)}>
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
        </svg>
        <span>장바구니 추가</span>
      </button>
    </div>
  </div>
  <div class='product-detail'>

  </div>
</section>
{/if}
