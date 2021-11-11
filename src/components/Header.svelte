<script>
import { link, navigate } from 'svelte-navigator';
import { onMount, createEventDispatcher } from 'svelte';
import { user } from '../store'
const dispatch = createEventDispatcher();
let search_term = "";

function toggleSidebar(){
  dispatch('sidebar-click');
}

function search(e){
  if(e.keyCode===13){
    if(search_term.length > 0){
      navigate(`/products/search/${search_term}/`);
    }else{
      alert('검색어를 입력해주세요!')
    }
  }
}
</script>
<header >
  <div class={`py-4 shadow-sm ${$user && $user.is_staff ? "bg-gray-800 text-white" : ""}`}>
    <nav class="lg:max-w-5xl mx-auto flex justify-between items-center">
      <a href='/products' class='font-bold text-2xl flex items-center ' use:link><span>ecommerce</span>{#if $user && $user.is_staff}<span class='ml-2 text-base text-gray-200'>admin</span>{/if}</a>
      
      <ul class='flex items-center space-x-4'>
        {#if $user && !$user.is_staff}
        <li>
          <div class='bg-gray-100 rounded-lg flex px-3 py-2'>
            <input on:keypress={search} bind:value="{search_term}" type='text' placeholder="Search for products" spellcheck={false} class='flex-auto bg-gray-100 focus-visible:outline-none mr-1' />
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
            </svg>
          </div>
        </li>
        <li>
          <button on:click={()=>navigate('/cart/')}>
            <!-- {#if cart_items.unread_count }
            <span class='bg-red-600 absolute top-2 right-3 px-1 py-1 rounded-full'></span>
            {/if} -->
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
            </svg>
          </button>
        </li>
        {/if}
        <li>
          <button on:click="{toggleSidebar}"> 
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
            </svg>
          </button>
        </li>
      </ul>
    </nav>
  </div>
  
</header>