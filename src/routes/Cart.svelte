<script>
import { onMount } from "svelte";
import api from "../utils/api";
import { user } from '../store';
import { DEFAULT_SHIPPING_FEE } from '../utils/common'
import NumberSelector from "../components/NumberSelector.svelte";

export let navigate;
let cartItems = [];
let totalProductPriceAmount = 0;
let orderShippingFee = DEFAULT_SHIPPING_FEE;

$:{
  totalProductPriceAmount = calTotalProductAmount(cartItems);
  orderShippingFee = totalProductPriceAmount && DEFAULT_SHIPPING_FEE
}

function calTotalProductAmount(items){
  let result = 0
  for(const item of items){
    result += item.option.price * item.qty;
  }
  return result;
}

onMount(async ()=> {
  try {
    const res = await api.get(`/cart/`);
    cartItems = res.data
  } catch (error) {
    console.error(error)    
  }
});
async function handleQtyChange(item){
  const { id, qty } = item;
  try{
    const res = await api.patch(`/cart/`, {id, qty});
  }catch(error){
    console.error(error);
  }
}
async function handleDeleteFromCart(cartItemIds){
  if(confirm('해당 장바구니 품목을 삭제하시겠습니까?')){
    try {
      // data = JSON.stringify(data)
      const res = await api.put(`/cart/`, cartItemIds);
      cartItems = cartItems.filter(item => !cartItemIds.includes(item.id))
    } catch (error) {
      console.error(error) 
    }
  }
}
async function handleOrder(mockFail){

  if(cartItems.length < 1){
    alert('장바구니가 비었습니다.');
    return;
  }

  let data = {payment_method: 'kakaopay'}

  if(mockFail){
    data['mock_fail'] = true
  }

  try {
    let res = await api.post(`/cart/checkout/`);
    let order = res.data['order']
    res = await api.post(`/orders/${order.id}/proceed_payment/`, data);
    navigate(`/orders/${order.id}/`);
  } catch (error) {
    console.error(error.response);
    navigate(`/orders/${error.response.data.detail.order}/`);
  }
}

</script>
<section class='lg:max-w-5xl mx-auto h-full py-8'>
  <h1 class='text-3xl font-bold mb-4'>장바구니</h1>
  <div class='flex'>
    <div class='flex-auto'>
      <table class='w-full'>
        <thead>
          <tr>
              <th></th>
              <th>이름</th>
              <th>수량</th>
              <th>가격</th>
              <th></th>
          </tr>
        </thead>
        <tbody>
        {#each cartItems as item, index (index)}
          <tr class='items-center'>
            <td class='w-44'><img class="object-cover w-32 h-full" alt={item.product_image.name} src={item.product_image.file} /></td>
            <td>
              <div class='font-semibold text-xl'>
                {item.option.product.name}
              </div>
              <div class='text-gray-400'>
                {item.option.name}: {item.option.value}
              </div>
            </td>
            
            <td class='text-center'><NumberSelector on:change={async ()=>handleQtyChange(item)} bind:num="{item.qty}"/></td>
            <td class='text-right w-20'>{item.option.price * item.qty}원</td>
            <td class='w-20 text-center'>
              <button on:click="{()=>handleDeleteFromCart([item.id])}" class='px-2 py-2 border'>
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </button>
            </td>
          </tr>
        {:else}
          <tr>
            <td colspan='4' class='text-center py-10'>
              장바구니가 비었습니다.    
            </td>
          </tr>
        {/each}
        </tbody>
      </table>
    </div>
    <div class='w-52'>
      <div class='bg-gray-50 px-4 py-4'>
        <div class='flex justify-between mb-2'>
          <div>총 상품 금액</div>
          <div>{totalProductPriceAmount}원</div>
        </div>
        <div class='flex justify-between mb-8'>
          <div>배송비</div>
          <div>{orderShippingFee}원</div>
        </div>
        <div class='text-right mb-8'>
          <div class='font-semibold'>예상 결제 금액</div>
          <div class='font-semibold text-2xl'>{totalProductPriceAmount+orderShippingFee}원</div>
        </div>
        <button class='px-4 py-2 w-full bg-green-500 hover:bg-green-600 mb-2 text-gray-50 hover:text-white' on:click={async()=>handleOrder()}>전체상품 주문하기</button>
        <button class='px-4 py-2 w-full bg-gray-200 hover:bg-gray-300' on:click={async()=>handleOrder(true)}>주문 실패</button>
      </div>
    </div>
  </div>
</section>