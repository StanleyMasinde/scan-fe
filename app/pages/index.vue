<script setup lang="ts">
const videoPreview = ref<HTMLVideoElement | null>(null);
const isScanning = ref(false);
const detectedCode = ref<string | undefined>(undefined);
const errorMessage = ref<string | undefined>(undefined);

let stream: MediaStream | undefined = undefined;
let detector:
  | {
      detect: (
        source: ImageBitmapSource,
      ) => Promise<Array<{ rawValue?: string }>>;
    }
  | undefined = undefined;
let scanTimer: number | null = null;

const supportsBarcodeDetector =
  typeof window !== "undefined" && "BarcodeDetector" in window;

async function startScanner() {
  errorMessage.value = undefined;
  detectedCode.value = undefined;

  if (!supportsBarcodeDetector) {
    errorMessage.value = "Barcode detection is not supported in this browser.";
    return;
  }

  try {
    const isMobile = /Mobi|Android/i.test(navigator.userAgent);

    stream = await navigator.mediaDevices.getUserMedia({
      video: {
        facingMode: "environment",
        width: isMobile
          ? { ideal: 720 }
          : { min: 1024, ideal: 1280, max: 1920 },
        height: isMobile
          ? { ideal: 1280 }
          : { min: 576, ideal: 720, max: 1080 },
      },
      audio: false,
    });

    if (!videoPreview.value) {
      errorMessage.value = "Video element not available.";
      return;
    }

    videoPreview.value.srcObject = stream;
    await videoPreview.value.play();

    const DetectorCtor = (
      window as typeof window & { BarcodeDetector: new (...args: any[]) => any }
    ).BarcodeDetector;
    detector = new DetectorCtor({
      formats: [
        "aztec",
        "ean_13",
        "code_128",
        "code_39",
        "codabar",
        "qr_code",
        "upc_a",
        "upc_e",
      ],
    });

    isScanning.value = true;
    scanLoop();
  } catch (error) {
    errorMessage.value = "Unable to access camera. Check permissions.";
    console.error(error);
  }
}

function stopScanner() {
  isScanning.value = false;

  if (scanTimer !== null) {
    window.clearTimeout(scanTimer);
    scanTimer = null;
  }

  if (videoPreview.value) {
    videoPreview.value.pause();
    videoPreview.value.srcObject = null;
  }

  if (stream) {
    stream.getTracks().forEach((track) => track.stop());
    stream = undefined;
  }
}

async function scanLoop() {
  if (!isScanning.value || !videoPreview.value || !detector) {
    return;
  }

  try {
    const barcodes = await detector.detect(videoPreview.value);

    const firstMatch = barcodes[0];
    if (firstMatch?.rawValue) {
      detectedCode.value = firstMatch.rawValue;
      stopScanner();
      return;
    }
  } catch (error) {
    // Ignore occasional frame-read errors while camera is warming up.
    console.debug("scan loop warning", error);
  }

  scanTimer = window.setTimeout(scanLoop, 250);
}

onBeforeUnmount(() => {
  stopScanner();
});
</script>

<template>
  <div class="flex items-center-safe border">
    <div class="w-full m-10">
      <!-- Preview -->
      <video
        class="border rounded-lg w-full object-cover"
        ref="videoPreview"
        autoplay
        playsinline
      />
      <div class="text-center mt-2">
        <p>Tap Scan Now to begin</p>
      </div>

      <div>
        <button
          :disabled="isScanning"
          class="w-full bg-primary text-white py-2 rounded-lg disabled:bg-neutral disabled:text-gray-200"
          @click.prevent="startScanner()"
        >
          {{ isScanning ? "Scanning" : "Scan Now" }}
        </button>
      </div>

      <div class="mt-5 text-center">
        <div v-if="errorMessage">
          <p class="text-lg text-red-500">
            An error occoured <b>{{ errorMessage }}</b>
          </p>
        </div>
      </div>

      <div class="mt-5 text-center" v-if="detectedCode">
        <div>
          <p class="text-lg">
            EAN found! <b>{{ detectedCode }}</b>
          </p>
        </div>

        <div class="my-5">
          <NuxtLink
            class="py-2 px-3 my-5 bg-primary text-white rounded-lg"
            :to="`/products/${detectedCode}`"
            >View Product</NuxtLink
          >
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
