### Blueprint

```
nvidia_bluefield3:
	vendor: Nvidia
	platform: Linux
	hw_model: Bluefield B3240
	ports:
		- type: ethernet
		  speed: 400 gbps
		  hw_type: 400gbase-x-qsfp112
		  scheme: moai_phys
		  port_indexes: 3:4
		- type: eth
		  speed: 1 gbps
		  hw_type: 1000base-t
		  scheme: eth
		  port_indexes: 0
```

