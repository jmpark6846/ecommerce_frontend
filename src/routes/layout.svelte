<script>
import { link, useNavigate, useLocation } from 'svelte-navigator';
import Header from "../components/Header.svelte";
import { user } from '../store';
import api from '../utils/api'
const navigate = useNavigate();
const location = useLocation();

let isSidebarOpen = false;

$:{
  if(isSidebarOpen){
    document.body.style.setProperty("overflow", "hidden");
  }else{
    document.body.style.removeProperty("overflow");
  }
}

function toggleSidebar(){
  isSidebarOpen = !isSidebarOpen;
}

async function logout(){
  const res = await api.post('accounts/logout/');
  if(res.status === 200){
    user.set(null);
    isSidebarOpen = false;
    const from = ($location.state && $location.state.from) || "/";
    navigate(from, { replace: true });
  }else{
    throw new Error('로그아웃에 실패하였습니다.')
  }
}
</script>

<div class='flex flex-col h-full'>
  <Header on:sidebar-click="{toggleSidebar}"/>
  <div class={`${isSidebarOpen ? 'opacity-100':'opacity-0'} transition-opacity`} style="z-index:10000;">
    <div 
      on:click="{()=>isSidebarOpen=false}"
      class={`absolute inset-0 bg-black opacity-50  ${isSidebarOpen ? 'block':'hidden'}`} tabindex="0" >
    </div>
  </div>

  <aside id='sidebar' class={`${isSidebarOpen ? 'translate-x-0':'translate-x-full'} py-4 transform fixed w-60 shadow h-full bg-white overflow-auto ease-in-out transition-all duration-300 top-0 right-0`} style="z-index:10001;">
    <div class='flex justify-start'>
      <div class='px-4 py-2'>
        <button on:click="{toggleSidebar}">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
          </svg>
        </button>
      </div>
    </div>
    <div class='py-4'>
      <div class='px-4'>  
        {#if $user }
          <button on:click="{async ()=> await logout()}">
            로그아웃
          </button>  
        {:else}
          <a href='/login/' on:click={()=>isSidebarOpen=false} >로그인</a>
        {/if}
      </div>
    </div>
    {#if $user && !$user.is_staff}

    <div class='py-4'>
      <div class='px-4'>
        <a href='/products/' on:click={()=>isSidebarOpen=false} use:link>상품보기</a>
      </div>
    </div>
    <div class='py-4'>
      <div class="px-4">
        <a href='/orders/' on:click={()=>isSidebarOpen=false} use:link>주문 내역</a>
      </div>
    </div>
    {/if}
  </aside>

  <main class='flex-auto'>
    <slot />
  </main>
  <!-- <Footer /> -->
</div>