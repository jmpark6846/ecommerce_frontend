<script>
import { Router, Route } from 'svelte-navigator'
import dayjs from 'dayjs'
import 'dayjs/locale/ko';
import relativeTime from 'dayjs/plugin/relativeTime';
import { onMount } from 'svelte';
import { user, cart_items } from './store';

import api from './utils/api';
import Layout from './routes/layout.svelte';
import Index from './routes/index.svelte';
import Login from './routes/login.svelte';
import NotFound404 from './routes/NotFound404.svelte';
import ProductList from './routes/products/ProductList.svelte';
import ProductDetail from './routes/products/ProductDetail.svelte';
import PrivateRoute from './components/PrivateRoute/PrivateRoute.svelte';
import Cart from './routes/Cart.svelte';
import OrderDetail from './routes/orders/OrderDetail.svelte';
import OrderList from './routes/orders/OrderList.svelte';
import ProductSearch from './routes/products/ProductSearch.svelte';
import AdminPage from './routes/admin/AdminDashboard.svelte';
import AdminDashboard from './routes/admin/AdminDashboard.svelte';

dayjs.locale('ko');
dayjs.extend(relativeTime);
api.defaults.baseURL = BASE_URL;

onMount(async ()=> {
  if($user){
		try {
			const res = await api.get('accounts/user/');
			user.set(res.data)
		} catch (error) {
			console.error(error)	
		}
	}
});
</script>

<Router>
	<Layout>
		<PrivateRoute path="admin/" let:navigate>
			<AdminDashboard {navigate}/>
		</PrivateRoute>
		
		<PrivateRoute path="orders/:order_id/" let:navigate>
			<OrderDetail {navigate}/>
		</PrivateRoute>
		<PrivateRoute path="orders/" let:navigate>
			<OrderList {navigate}/>
		</PrivateRoute>
		<PrivateRoute path="products/search/:search_term/" let:params let:navigate>
			<ProductSearch search_term={params.search_term} {navigate}/>
		</PrivateRoute>
		<PrivateRoute path="products/:product_id/" let:navigate>
			<ProductDetail {navigate}/>
		</PrivateRoute>
		<PrivateRoute path="products/" let:navigate>
			<ProductList {navigate}/>
		</PrivateRoute>
		<PrivateRoute path="cart/" let:navigate>
			<Cart {navigate}/>
		</PrivateRoute>
		
		<Route path='/login/' let:navigate>
			<Login {navigate} />
		</Route>
		<Route path='/' component={Index} let:navigate/>
		<Route path='*' component={NotFound404}/>
	</Layout>
</Router>
<style global> 
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";
@import "./global.css";


.page-nav :global(.option){
	@apply px-2 py-1 cursor-pointer text-gray-700 hover:text-gray-900 hover:bg-gray-100 focus:bg-gray-300 rounded-md mr-1 text-center;
	line-height: initial;
}

.page-nav :global(.option.active){
	@apply  bg-gray-300 text-gray-600 hover:bg-gray-300 hover:text-gray-700; 
}

.page-nav :global(.pagination-nav){
	margin-top: 8px;
	text-align: center;
}


</style>