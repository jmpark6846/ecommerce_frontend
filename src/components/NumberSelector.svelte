<script>
import { createEventDispatcher } from 'svelte';
export let num;

const dispatch = createEventDispatcher();

function handleDecrease(){
  if(num > 1){
    num -= 1;
  }
  dispatch('change');
}
function handleIncrease(){
  num += 1;
  dispatch('change');
}


</script>
<div class='border border-gray-300 inline-block'>
  <button  class='px-3 py-1 bg-gray-100 hover:bg-gray-200' on:click="{handleDecrease}">-</button>
  <input on:change disabled class='px-3 py-1 w-16 disabled:bg-white text-center focus:ring-2 focus:outline-none' bind:value={num}/>
  <button  class='px-3 py-1 bg-gray-100 hover:bg-gray-200' on:click="{handleIncrease}">+</button>
</div>