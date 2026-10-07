ifeq ($(CONFIG_BUILD_ARM64_DT_OVERLAY),y)
dtbo-$(CONFIG_ARCH_LITO) += sm7250-MIZUKI00-rev10-overlay.dtbo \
                            sm7250-MIZUKI00-rev17-overlay.dtbo \
                            sm7250-MIZUKI00-rev50-overlay.dtbo \
                            sm7250-MIZUKI02-rev00-overlay.dtbo \
                            sm7250-MIZUKI03-rev10-overlay.dtbo \
                            sm7250-MIZUKI03-rev60-overlay.dtbo \
                            sm7250-MIZUKI04-rev00-overlay.dtbo \
                            sm7250-MIZUKI04-rev10-overlay.dtbo \
                            sm7250-MIZUKI04-rev12-overlay.dtbo \
                            sm7250-MIZUKI04-rev50-overlay.dtbo

sm7250-MIZUKI00-rev10-overlay.dtbo-base := lito.dtb
sm7250-MIZUKI00-rev17-overlay.dtbo-base := lito.dtb
sm7250-MIZUKI00-rev50-overlay.dtbo-base := lito.dtb
sm7250-MIZUKI02-rev00-overlay.dtbo-base := lito.dtb
sm7250-MIZUKI03-rev10-overlay.dtbo-base := lito.dtb
sm7250-MIZUKI03-rev60-overlay.dtbo-base := lito.dtb
sm7250-MIZUKI04-rev00-overlay.dtbo-base := lito.dtb
sm7250-MIZUKI04-rev10-overlay.dtbo-base := lito.dtb
sm7250-MIZUKI04-rev12-overlay.dtbo-base := lito.dtb
sm7250-MIZUKI04-rev50-overlay.dtbo-base := lito.dtb
endif

always      := $(dtb-y)
subdir-y    := $(dts-dirs)
clean-files := *.dtb *.dtbo

