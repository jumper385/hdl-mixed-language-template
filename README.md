# Yosys + GHDL Mixed Language Template

## Introduction

This template enables you to synthesize mixed-language HDL projects (VHDL + Verilog) targeting ICE40 FPGAs. It demonstrates how to seamlessly integrate VHDL and Verilog modules in a single design using modern open-source synthesis tools.

### Goals
- **Mixed Language Support**: Combine VHDL and Verilog in a single project
- **Open Source Toolchain**: Use only free, open-source FPGA tools
- **Automated Synthesis**: GitHub Actions CI/CD for continuous integration
- **Extensible Design**: Easy template for starting new mixed-language projects

### Backend Technologies
- **[Yosys](https://yosyshq.net/yosys/)**: Open-source synthesis suite for Verilog and VHDL
- **[GHDL](https://github.com/ghdl/ghdl)**: Open-source VHDL simulator and synthesizer
- **[GHDL-Yosys Plugin](https://github.com/ghdl/ghdl-yosys-plugin)**: Bridge between GHDL and Yosys for VHDL synthesis
- **[nextpnr](https://github.com/YosysHQ/nextpnr)**: Portable FPGA place-and-route tool for ICE40
- **[IceStorm](https://github.com/YosysHQ/icestorm)**: Bitstream generation tools for Lattice ICE40 FPGAs

### Build Automation
The project includes automated GitHub Actions workflows that:
- Trigger on HDL file changes (`.vhd`, `.vhdl`, `.v`)
- Use containerized synthesis environment (`hdlc/ghdl:yosys`)
- Generate synthesis artifacts (JSON, ASC, BIN files)
- Upload build artifacts for download

## Project Structure

```
├── .github/workflows/
│   └── synth.yml           # GitHub Actions CI workflow
├── scripts/
│   └── ice40up5k_synth.ys  # Yosys synthesis script
├── vhdl/
│   └── top.vhd            # VHDL top-level entity
├── verilog/
│   ├── and2.v             # Verilog AND gate module
│   └── or2.v              # Verilog OR gate module
├── io.pcf                 # Pin constraint file for ICE40UP5K
└── README.md              # This file
```

### Current Design
The example design implements a simple logic function `d_o = (a_i & b_i) | c_i`:
- **VHDL Top Entity** (`vhdl/top.vhd`): Main design entity that instantiates Verilog components
- **Verilog Components**: Basic logic gates (`and2.v`, `or2.v`) implemented in Verilog
- **Mixed Integration**: VHDL entity instantiates Verilog modules using component declarations

## Adding Files to the Build

### Adding VHDL Files
1. Place your VHDL files in the `vhdl/` directory
2. Update the synthesis script `scripts/ice40up5k_synth.ys`:
   ```tcl
   ghdl --std=08 -fsynopsys -frelaxed-rules \
       vhdl/top.vhd \
       vhdl/your_new_file.vhd \  # Add your file here
       -e top;
   ```

### Adding Verilog Files
1. Place your Verilog files in the `verilog/` directory
2. Update the synthesis script `scripts/ice40up5k_synth.ys`:
   ```tcl
   read_verilog \
       verilog/and2.v \
       verilog/or2.v \
       verilog/your_new_module.v  # Add your file here
   ```

### Updating Pin Constraints
Modify `io.pcf` to match your design's I/O:
```
set_io signal_name pin_number
```

For ICE40UP5K-SG48 package pin mapping, refer to the [ICE40 Family Data Sheet](https://www.latticesemi.com/Products/FPGAandCPLD/iCE40Ultra).

## Manual Build Requirements

### System Dependencies
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y \
    yosys \
    ghdl \
    nextpnr-ice40 \
    fpga-icestorm \
    ghdl-yosys-plugin

# Fedora/RHEL
sudo dnf install -y \
    yosys \
    ghdl \
    nextpnr \
    icestorm

# Arch Linux
sudo pacman -S \
    yosys \
    ghdl \
    nextpnr \
    icestorm
```

### Docker Alternative
Use the same container as the CI workflow:
```bash
docker run --rm -v $(pwd):/work -w /work hdlc/ghdl:yosys \
    bash -c "apt update && apt install nextpnr-ice40 fpga-icestorm -y && yosys -s scripts/ice40up5k_synth.ys"
```

## Manual Build Instructions

### 1. Verify Tool Installation
```bash
yosys -V
ghdl --version
nextpnr-ice40 --version
icepack --help
```

### 2. Run Synthesis
```bash
# Execute the complete synthesis flow
yosys -s scripts/ice40up5k_synth.ys
```

This command will:
1. Load the GHDL plugin for VHDL support
2. Read all Verilog source files
3. Analyze and elaborate VHDL files with GHDL
4. Synthesize to ICE40 primitives
5. Perform place-and-route with nextpnr
6. Generate the final bitstream

### 3. Generated Outputs
- `top.json`: Post-synthesis netlist
- `top.asc`: Post-place-and-route ASCII file
- `ice40up5k_top.bin`: Final bitstream ready for programming

### 4. Programming the FPGA
```bash
# Using iceprog (if you have compatible hardware)
iceprog ice40up5k_top.bin

# Or convert to other formats as needed
icepack top.asc top.bin
```

## Extending to Other FPGAs

To target different FPGA families:
1. Replace `synth_ice40` with appropriate synthesis command (`synth_xilinx`, `synth_intel`, etc.)
2. Update place-and-route tool (`nextpnr-xilinx`, `vivado`, etc.)
3. Modify pin constraints file format
4. Update bitstream generation commands

## Troubleshooting

### Common Issues
- **GHDL Plugin Not Found**: Ensure `ghdl-yosys-plugin` is installed
- **Synthesis Errors**: Check VHDL/Verilog syntax and component/module names match
- **Place-and-Route Fails**: Verify pin constraints and design doesn't exceed FPGA resources

### Getting Help
- [Yosys Documentation](https://yosyshq.readthedocs.io/)
- [GHDL Documentation](https://ghdl.readthedocs.io/)
- [ICE40 Tools](https://github.com/YosysHQ/icestorm)

## License

This template is provided as-is for educational and development purposes.
