<script>
import { onMount } from 'svelte';
import Chart from 'chart.js/auto';
import ChartDataLabels from 'chartjs-plugin-datalabels';
import api from '../../utils/api';
import dayjs from 'dayjs'
let loading = true;
Chart.register(ChartDataLabels);
Chart.defaults.set('plugins.datalabels',{
  anchor: 'end',
  align: 'top',
})

const chart_data = {
  labels:[],
  datasets: [{  
    label:'',
    backgroundColor: 'rgb(255, 99, 132)',
    borderColor: 'rgb(255, 99, 132)',
    data: [],
    tension: 0.1
  }]
}

const weeks_chart_config = {
  type: 'bar',
  data: null,
  options: {
    responsive: true,
    plugins:{
      tooltip: {
        enabled:false
      },
      title: {
        text: '이번주 요일별 매출',
        display: false,
        font: {
          size:18
        }
      }

    }
    
  },
};
let weeksChart;
let top_selling_items=[];
function prepareForWeeksChart(data){
  let labels = [];
  let weeks_data = [];
  let week_data_map = {}

  data.labels.forEach(date => labels.push(dayjs(date).format('YYYY-MM-DD')));

  for(const week of data.qs){
    week_data_map[week['date']] = week['total_amount']
  }

  for(const label of labels){
    if(label in week_data_map){
      weeks_data.push(week_data_map[label])
    }else{
      weeks_data.push(0)
    }
  }
  chart_data.labels = labels;
  chart_data.datasets[0].data = weeks_data;
  weeks_chart_config.data = chart_data;
}

onMount(async ()=>{
  try {
    let res = await api.get('/ecommerce_admin/payments/weeks/')
    prepareForWeeksChart(res.data)   
    weeksChart = new Chart(
      document.getElementById('weeks_chart'),
      weeks_chart_config
    );

    res = await api.get('/ecommerce_admin/payments/top_selling_items/')
    console.log(res.data)
    top_selling_items = res.data
    loading=false;
  } catch (error) {
    console.error(error)
  }
})

</script>
<section class='lg:max-w-5xl mx-auto h-full py-8'>
  <h1 class='text-3xl font-bold mb-4'>대시보드</h1>

  <div class="flex  space-x-8">
    <div class='w-1/2'>
      <h2 class='text-2xl font-semibold mb-4'>이번주 매출 현황</h2>
      <div>
        <canvas id="weeks_chart"></canvas>
      </div>  
    </div>
    
    
    <div class='w-1/2'>
      <h2 class='text-2xl font-semibold mb-4'>이번달 매출 상위 제품</h2>
      <div>
        {#each top_selling_items as item, index (index)}
          <div class='flex items-center mb-4 justify-between'>
            <div class='flex items-center'>
              <div class='w-8'>{index+1}</div>
              <div class='w-24'>
                <img class="object-cover w-24 h-full" alt={item.option.name} src={item.option.image.file} />
              </div>
              <div class='px-4'>
                <span class='font-semibold mr-1'>{item.option.product.name}</span>
                <span class='text-gray-400'>{item.option.name}: {item.option.value}</span>
              </div>
            </div>
            
            <div class='flex items-center'>
              <div class='text-center mr-1'>{item.total_qty}개</div>
              <div class='text-right w-20'>{item.total_amount}원</div>
            </div>
          </div>
        {/each}
      </div>
    </div>  
  </div>
</section>
