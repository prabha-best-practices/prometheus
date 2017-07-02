### To find the number of time series stored by Prometheus

+ storage.local.path folder :
`ls -l {{0..9},{a..f}}{{0..9},{a..f}} | grep -E "*.db$" | wc -l`

+ Prometheus console : `({name=~".+"})`
