# 1. When using oracles that aggregate prices from sources, check the heartbeat time of the provider so the data remains fresh and not stale, also don't use the same heartbeat for different tokens that have different update timer (check chainlink)

# 2. When switching pools in Aave, always check for approval for the new pool else, transactions involving approvals and withdrawals will fail