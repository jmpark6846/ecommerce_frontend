<script>
import { onMount } from "svelte";
import dayjs from 'dayjs'
import { link } from 'svelte-navigator';
import { PaginationNav } from 'svelte-paginate'
import api from "../../utils/api";
export let navigate;

let orderInfo = {};
let orders = [];
const ORDER_STATUS = {
  0: "주문 접수",
  1: "결제 완료",
  2: "결제 실패",
  3: "배송 준비",
  4: "배송 완료",
  5: "주문 취소"
}

onMount(async ()=> {
  await fetchOrder();
});

async function fetchOrder(page_num=1){
  try {
    const res = await api.get(`/orders/?page_num=${page_num}`);
    orders = res.data.list;
    delete res.data['list'];
    orderInfo = res.data;
  } catch (error) {
    console.error(error)    
  }
}

</script>

<section class='lg:max-w-5xl mx-auto h-full py-8'>
  <h1 class='text-3xl font-bold mb-4'>주문 내역</h1>
  <div class='flex'>
    <div class='flex-auto'>
      <table class='w-full'>
        <thead>
          <tr class='border-b border-gray-300' style='border-top: 1px black solid;'>
              <th class='py-4'>주문번호</th>
              <th>상품</th>
              <th>날짜</th>
              <th>가격</th>
              <th>상태</th>
              <th></th>
          </tr>
        </thead>
        <tbody>
        {#each orders as order, index (index)}
          <tr class='items-center'>
            <td class='py-4 w-32 text-center'>{order.id}</td>
            <td class=''>
              {#each order.items as item, index (index)}   
              <a href={`/orders/${order.id}/`} use:link>
                <div>
                  <span class='mr-1'>
                    {item.option.product.name}
                  </span>
                  <span class='text-gray-400'>
                    {item.option.name}: {item.option.value}
                  </span>
                </div>
              </a>           
              {/each}
            </td>
            <td class='w-40'>
              <span class='ml-2 text-sm'>{dayjs(order.ordered_at).format('YYYY.MM.DD  A h:mm')}</span>
            </td>
            <td class='text-right w-32'>{order.total_amount}원</td>
            <td class='text-center w-32'>{ORDER_STATUS[order.status]}</td>
            <td></td>
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
      <div class="page-nav">
        <PaginationNav
          totalItems="{orderInfo.count}"
          pageSize="{orderInfo.page_size}"
          currentPage="{orderInfo.current_page}"
          limit="{2}"
          showStepOptions="{false}"
          on:setPage="{async (e) => await fetchOrder(e.detail.page)}"
        />
      </div>
    </div>
  </div>
</section>

