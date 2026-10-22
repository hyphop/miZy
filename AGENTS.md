# H2+ kernel upgrade

This fork tracks a kernel upgrade for an Orange Pi Zero H2+ with 512 MiB RAM and
16 MiB SPI flash. Preserve miZy, working XR819 Wi-Fi, and the HTTPS CONNECT proxy.
Replacing the whole system with Armbian is not the requested outcome.

Read `docs/kernel-upgrade/STATUS.md` before device work. Record actual boot and
network test results; distinguish downloaded candidates from tested kernels.

Keep firmware dumps, provisioned images, Wi-Fi credentials, SSH keys, and VPN
state out of git. Local device artifacts live outside this checkout. Retain a
verified full SPI backup before flash changes. The existing bootloader and UART
are the recovery path; do not assume a TF card is inserted.

The `kernel` submodule points to the user's fork of the upstream miZy kernel
repository. Changes to that repository require their own commits before updating
the submodule pointer here.
