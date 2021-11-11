<script>
  import { useNavigate, useLocation } from "svelte-navigator";
  import user from "../../store/user";

  const navigate = useNavigate();
  const location = useLocation();

  $: if (!$user) {
    navigate("/login", {
      state: { from: $location.pathname }
    })
  }else if($user.is_staff){
    navigate("/admin/");
  }
</script>

{#if $user}
  <slot />
{/if}
