# All output stays outside the checkout. Reuse the fork's macOS 15 graphics layer.
WORK ?= /tmp/Tomodachi_build/host
OUT := $(WORK)/build
BACKEND ?= metal3
ifeq ($(BACKEND),metal4)
BACKEND_FLAGS := -mmacosx-version-min=26.0 -DDM_NVN_M3=0
BINARY := $(OUT)/tomodachi-native-metal4
GFX_OUT := $(OUT)/metal4/gfx
else ifeq ($(BACKEND),metal3)
BACKEND_FLAGS := -mmacosx-version-min=15.0 -DDM_NVN_M3=1
BINARY := $(OUT)/tomodachi-native
GFX_OUT := $(OUT)/gfx
else
$(error BACKEND must be metal3 or metal4)
endif
DONOR := ../totk
GFX := $(DONOR)/third_party/d2r_gfx
GFX_SRC := $(wildcard $(GFX)/gfx/nvn/*.c $(GFX)/gfx/nvn/*.m $(GFX)/gfx/nvn/*.S $(GFX)/gfx/tex/*.c $(GFX)/gfx/tex/*.m)
GFX_OBJ := $(patsubst $(GFX)/%,$(GFX_OUT)/%.o,$(GFX_SRC))
RUNTIME := $(filter-out $(DONOR)/runtime/loader.c $(DONOR)/runtime/game_patches.c $(DONOR)/runtime/game_patches_v143.c $(DONOR)/runtime/world_survey.c $(DONOR)/runtime/autoload.c $(DONOR)/runtime/frontend_ios.m,$(wildcard $(DONOR)/runtime/*.c $(DONOR)/runtime/*.m $(DONOR)/runtime/*.S))
TITLE_RUNTIME := $(wildcard runtime/*.c runtime/*.m)
FLAGS := -O2 -g $(BACKEND_FLAGS) -DDM_NVN_SERIAL_REPLAY=1 -DNATIVE_GAME_TITLE='"Tomodachi Life: Living the Dream"' -DNATIVE_APPLICATION_ID=0x010051F0207B2000ULL -I$(DONOR)/runtime -I$(GFX)/gfx/nvn -I$(GFX)/gfx/nvn/include -I$(OUT) -I/opt/homebrew/opt/opus/include/opus -I/opt/homebrew/opt/libvpx/include
FRAMEWORKS := Metal MetalFX Foundation QuartzCore ImageIO CoreGraphics CoreText UniformTypeIdentifiers Cocoa CoreVideo CoreFoundation AudioToolbox GameController CoreHaptics IOKit
.PHONY: all graphics check-toolchain
all: $(BINARY)
graphics: $(GFX_OBJ)
$(GFX_OUT)/%.o: $(GFX)/% $(wildcard $(GFX)/gfx/nvn/*.h $(GFX)/gfx/nvn/include/*.h) Makefile | check-toolchain
	@mkdir -p "$(dir $@)"
	xcrun clang -std=gnu11 $(FLAGS) -fobjc-arc -Wno-unused-function -c "$<" -o "$@"
$(BINARY): $(TITLE_RUNTIME) $(RUNTIME) $(wildcard $(DONOR)/runtime/*.h runtime/*.h) $(OUT)/runtime_tables.h $(OUT)/romfs_config.h $(OUT)/shared_fonts_config.h $(OUT)/title_config.h $(GFX_OBJ) Makefile | check-toolchain
	xcrun clang -std=c11 $(FLAGS) -Wno-deprecated-declarations -fobjc-arc $(TITLE_RUNTIME) $(RUNTIME) $(GFX_OBJ) /opt/homebrew/opt/opus/lib/libopus.a /opt/homebrew/opt/libvpx/lib/libvpx.a $(addprefix -framework ,$(FRAMEWORKS)) -o "$@"

check-toolchain:
ifeq ($(BACKEND),metal4)
	@python3 tools/check_metal4_sdk.py
endif
