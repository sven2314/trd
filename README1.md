# trd
cd vck190-base-trd
make sdcard  PFM=vck190_mipiRxQuad_hdmiTx OVERLAY=filter2d
overlays/filter2d/kernels

make all -C filter2d_combined/ PLATFORM=../../../platforms/xilinx_vck190_mipiRxQuad_hdmiTx_202210_1/vck190_mipiRxQuad_hdmiTx.xpfm
