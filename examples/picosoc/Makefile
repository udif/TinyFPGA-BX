PROJTOP = hardware

PIN_DEF = $(PROJTOP).pcf

DEVICE = lp8k
PACKAGE = cm81

CLK_MHZ = 16

GCC = /opt/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin/riscv64-unknown-elf-gcc
OBJCOPY = /opt/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin/riscv64-unknown-elf-objcopy

RTL_USB_DIR = tinyfpga_bx_usbserial/usb

USB_SOURCES = \
	$(RTL_USB_DIR)/edge_detect.v \
	$(RTL_USB_DIR)/serial.v \
	$(RTL_USB_DIR)/usb_fs_in_arb.v \
	$(RTL_USB_DIR)/usb_fs_in_pe.v \
	$(RTL_USB_DIR)/usb_fs_out_arb.v \
	$(RTL_USB_DIR)/usb_fs_out_pe.v \
	$(RTL_USB_DIR)/usb_fs_pe.v \
	$(RTL_USB_DIR)/usb_fs_rx.v \
	$(RTL_USB_DIR)/usb_fs_tx_mux.v \
	$(RTL_USB_DIR)/usb_fs_tx.v \
	$(RTL_USB_DIR)/usb_reset_det.v \
	$(RTL_USB_DIR)/usb_serial_ctrl_ep.v \
	$(RTL_USB_DIR)/usb_uart_bridge_ep.v \
	$(RTL_USB_DIR)/usb_uart_core.v \
	$(RTL_USB_DIR)/usb_uart_i40.v \
 	pll.v

USB_CLK_MHZ = 48

all: upload

pll.v:
	icepll -i 16 -o $(USB_CLK_MHZ) -m -f $@

upload: $(PROJTOP).bin firmware.bin
	tinyprog -p $(PROJTOP).bin -u firmware.bin


$(PROJTOP).json: $(PROJTOP).v spimemio.v simpleuart.v picosoc.v picorv32.v $(USB_SOURCES)
	yosys -ql $(PROJTOP).log -p 'synth_ice40 -top $(PROJTOP) -json $(PROJTOP).json' $^

%.asc: $(PIN_DEF) %.json usb_clk.py
	nextpnr-ice40 -l nextpnr.log --$(DEVICE) --placer-budgets --freq $(CLK_MHZ) --opt-timing --pre-pack usb_clk.py --package $(PACKAGE) --pcf $(PIN_DEF) --json $*.json --asc $@

gui: $(PIN_DEF) $(PROJTOP).json
	nextpnr-ice40 --$(DEVICE) --package $(PACKAGE) --pcf $(PIN_DEF) --json $(PROJTOP).json --asc $(PROJTOP).asc --gui

$(PROJTOP).bin: $(PROJTOP).asc
	icetime -d hx8k -c 12 -mtr $(PROJTOP).rpt $(PROJTOP).asc
	icepack $(PROJTOP).asc $(PROJTOP).bin


firmware.elf: sections.lds start.S firmware.c 
	$(GCC) -march=rv32imc -mabi=ilp32 -nostartfiles -Wl,-Bstatic,-T,sections.lds,--strip-debug,-Map=firmware.map,--cref  -ffreestanding -nostdlib -o firmware.elf start.S firmware.c

firmware.bin: firmware.elf
	$(OBJCOPY) -O binary firmware.elf /dev/stdout > firmware.bin


clean:
	rm -f firmware.elf firmware.hex firmware.bin firmware.o firmware.map \
	      $(PROJTOP).blif $(PROJTOP).log $(PROJTOP).asc $(PROJTOP).rpt $(PROJTOP).bin pll.v




