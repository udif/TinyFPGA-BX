PIN_DEF = hardware.pcf

DEVICE = lp8k
PACKAGE = cm81

CLK_MHZ = 24

GCC = /opt/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin/riscv64-unknown-elf-gcc
OBJCOPY = /opt/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin/riscv64-unknown-elf-objcopy

upload: hardware.bin firmware.bin
	tinyprog -p hardware.bin -u firmware.bin


hardware.json: hardware.v spimemio.v simpleuart.v picosoc.v picorv32.v
	yosys -ql hardware.log -p 'synth_ice40 -top hardware -json hardware.json' $^

%.asc: $(PIN_DEF) %.json
	nextpnr-ice40 -l nextpnr.log --$(DEVICE) --freq $(CLK_MHZ) --opt-timing --package $(PACKAGE) --pcf $(PIN_DEF) --json $*.json --asc $@

hardware.bin: hardware.asc
	icetime -d hx8k -c 12 -mtr hardware.rpt hardware.asc
	icepack hardware.asc hardware.bin


firmware.elf: sections.lds start.S firmware.c 
	$(GCC) -march=rv32imc -mabi=ilp32 -nostartfiles -Wl,-Bstatic,-T,sections.lds,--strip-debug,-Map=firmware.map,--cref  -ffreestanding -nostdlib -o firmware.elf start.S firmware.c

firmware.bin: firmware.elf
	$(OBJCOPY) -O binary firmware.elf /dev/stdout > firmware.bin


clean:
	rm -f firmware.elf firmware.hex firmware.bin firmware.o firmware.map \
	      hardware.blif hardware.log hardware.asc hardware.rpt hardware.bin




