# 1. When using oracles that aggregate prices from sources, check the heartbeat time of the provider so the data remains fresh and not stale, also don't use the same heartbeat for different tokens that have different update timer (check chainlink)

# 2. When switching pools in Aave, always check for approval for the new pool else, transactions involving approvals and withdrawals will fail

# 3. When auditing code that use uniswap like code always check if k = x * y  is maintained  // This simple ratio allows (amount_in * reserve_to) / reserve_from,// which does not maintain the k = x * y constant return ((amount * IERC20(to).balanceOf(address(this))) / IERC20(from).balanceOf(address(this)));, ALWAYS CHECK IF THE SWAP IN AND SWAP OUT GIVE EQUAL AMOUNTS 

((amount * IERC20(to).balanceOf(address(this))) / IERC20(from).balanceOf(address(this)));
10 in | 100 - 100 | 10 out (10*100/100 = 10)
20 in | 110 - 90  | 24 out (20*110/90 = 24)
24 in | 86  - 110 | 30 out (24*110/86 = 30)
30 in | 110  - 80 | 41 out (30*110/80 = 41)
41 in | 69  - 110 | 65 out (41*110/67 = 65)


