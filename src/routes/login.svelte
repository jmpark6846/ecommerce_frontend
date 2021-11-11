<script>
import { user } from "../store";
import api from "../utils/api";
export let navigate;

const demousers = [
  { email: 'demouser@mysite.com', password: 'demouser' },
  { email: 'root@mysite.com', password: 'root'}
]

async function login_demo(index=0){
  const res = await api.post('accounts/login/', demousers[index]);
  if(res.status === 200){
    user.set(res.data.user);
    if(res.data.user.is_staff){
      navigate("/admin/");
    }else{
      navigate("/products/");
    }
  }else{
    console.error(error);
  }
}


</script>

<section class='w-full h-full'>
  <div class='px-8 py-8 rounded-lg shadow mt-16 bg-white w-80 text-center mx-auto'>
    <h1 class='text-2xl font-semibold mb-4'>Sign In</h1>
    <button class='px-4 py-2 w-full mb-2 rounded-lg bg-green-500 hover:bg-green-600 text-white' on:click={()=>login_demo(0)}>데모 유저로 쇼핑몰 로그인</button>
    <button class='px-4 py-2 w-full rounded-lg bg-gray-700 hover:bg-gray-800 text-white' on:click={()=>login_demo(1)}>스태프로 어드민 로그인</button>
  </div>  
</section>

