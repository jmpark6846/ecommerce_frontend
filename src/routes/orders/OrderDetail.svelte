<script>
import { onMount } from "svelte";
import dayjs from 'dayjs'
import { useParams } from "svelte-navigator";
import api from "../../utils/api";
import { user } from '../../store'
export let navigate;

const params = useParams();

let order = {};
let orderItems = []
const ORDER_STATUS = {
  0: "주문 접수",
  1: "결제 완료",
  2: "결제 실패",
  3: "배송 준비",
  4: "배송 완료",
  5: "주문 취소"
}

onMount(async ()=> {
  try {
    const res = await api.get(`/orders/${$params.order_id}/`);
    order = res.data;
    orderItems = order['items']
  } catch (error) {
    console.error(error)    
  }
});

async function handleOrder(mockFail){
  let data = {payment_method: 'kakaopay'}

  if(mockFail){
    data['mock_fail'] = true
  }

  try {
    const res = await api.post(`/orders/${order.id}/proceed_payment/`, data);
    order = res.data['order']
    orderItems = order['items']
  } catch (error) {
    console.error(error);
    alert('결제 실패하였습니다.')
  }
}

</script>

<section class='lg:max-w-5xl mx-auto h-full py-8'>
  <h1 class='text-3xl font-bold mb-4'>주문 상세 보기 </h1>
  <div class='mb-4'>
    <span>주문일자</span>
    <span class='ml-2'>{dayjs(order.ordered_at).format('YYYY.MM.DD  A h:mm')}</span>
    <span class='text-center'>{ORDER_STATUS[order.status]}</span>
  </div>

  <div class='flex space-x-4'>
    <div class='flex-auto'>
      <table class='w-full'>
        <thead>
          <tr class='border-b border-gray-300' style='border-top: 1px black solid;'>
              <th class='py-4'>상품 주문번호</th>
              <th>상품</th>
              <th>수량</th>
              <th>가격</th>
          </tr>
        </thead>
        <tbody>
          {#each orderItems as item, index (index)}              
            <tr class='items-center'>
              <td class='text-gray-600 text-center w-32 py-4 px-4'>
                {item.id}
              </td>
              <td class='px-4'>
                <div>
                  <span class='mr-1'>
                    {item.option.product.name}
                  </span>
                  <span class='text-gray-400'>
                    {item.option.name}: {item.option.value}
                  </span>
                </div>
              </td>
              <td class='w-20 px-4 text-center'>{item.qty}개</td>
              <td class='w-32 px-4 text-right'>{item.amount}원</td>            
            </tr>
          {:else}
            <tr>
              <td colspan='4' class='text-center py-10'>
                주문 내역이 없습니다.
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
          <div>{order.products_amount}원</div>
        </div>
        <div class='flex justify-between mb-8'>
          <div>배송비</div>
          <div>{order.shipping_fee}원</div>
        </div>
        <div class='text-right mb-8'>
          <div class='font-semibold'>총 주문 결제 금액</div>
          <div class='font-semibold text-2xl'>{order.total_amount}원</div>
        </div>

        {#if [0,2].includes(order.status) }
          <button class='px-4 py-2 w-full bg-green-500 hover:bg-green-600 text-gray-50 hover:text-white' on:click={async()=>handleOrder()}>
            {order.status == 1 ? '결제하기' : '결제 재시도'}
          </button>
        {/if}
        
      </div>
    </div>
  </div>
</section>
