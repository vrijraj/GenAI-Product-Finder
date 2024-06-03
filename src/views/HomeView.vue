<template>
  <v-main
    :class="finalData != null && finalData.length ? 'mt-8' : 'fill-height'"
    class="ai-font pa-0"
  >
    <v-container
      fluid
      :class="finalData != null && finalData.length ? 'mt-4' : 'fill-height'"
      class="app-content-padding"
    >
      <v-row justify="center" align="center">
        <v-col md="12" cols="12" class="d-block d-sm-none d-md-none d-lg-none">
          <v-img :src="require('@/assets/logo.png')" width="250"></v-img>
          <p>
            Discover product details directly from videos and receive
            recommendations for similar items.
          </p>
        </v-col>
        <v-col md="7" cols="12" class="text-center">
          <video id="videoElement" class="video-camera" autoplay></video>
          <canvas v-show="false" width="1280" height="720" id="canvas"></canvas>
        </v-col>
        <v-col md="5" cols="12" class="pa-md-10">
          <div class="d-none d-sm-block d-md-block d-lg-block">
            <v-img :src="require('@/assets/logo.png')" width="250"></v-img>
            <p>
              Discover product details directly from videos and receive
              recommendations for similar items.
            </p>
          </div>
          <p>
            <b>Camera View:</b>
            {{ cameraView == "user" ? "Front Camera" : "Back Camera" }}
          </p>
          <v-btn
            dark
            class="mb-3 mr-3 search-btn"
            depressed
            rounded
            :loading="loading"
            @click="getData"
          >
            <v-icon left>mdi-image-search-outline</v-icon>
            Search for Product</v-btn
          >
         
          <v-btn
            color="#9949F9"
            class="mb-3 mr-3 d-md-none d-lg-none"
            @click="changeCamera"
            outlined
            dark
            rounded
            depressed
          >
            <v-icon left>mdi-camera-flip-outline</v-icon>
            Switch Camera</v-btn
          >
        </v-col>
      </v-row>
    </v-container>
    <v-container
      fluid
      v-if="finalData != null && finalData.length"
      transition="scroll-y-reverse-transition"
      class="text-left mt-md-10 mt-5 mt-sm-7 app-content-padding mb-10"
    >
      <v-row class="py-0">
        <v-col md="12" class="mb-0">
          <p class="mb-0" style="color: #242424; font-size: 110%">
            <b
              ><v-icon color="#242424">mdi-shopping-search-outline</v-icon>
              Results ({{ finalData.length }})</b
            >
          </p>
          <p style="font-size: 90%">
            This tool may display Inaccurate info, including product, Image and
            product model, double-check its responses.
          </p></v-col
        >
      </v-row>
      <v-row justify="start" align="start" class="pa-0 mt-0">
        <v-col
          md="4"
          sm="6"
          cols="12"
          v-for="(item, index) in finalData"
          :key="index"
          style="display: flex"
          class=""
        >
          <ProductCardVue v-if="Object.keys(item).length" :item="item" />
        </v-col>
      </v-row>
    </v-container>
  </v-main>
</template>

<script>
import {
  GoogleGenerativeAI,
  HarmCategory,
  HarmBlockThreshold,
} from "@google/generative-ai";
import ProductCardVue from "@/components/common/ProductCard.vue";
export default {
  name: "HomePage",
  data: () => ({
    loading: false,
    res: null,
    finalData: null,
    cxKey: process.env.VUE_APP_CX_ID,
    searchKey: process.env.VUE_APP_SEARCH_API,
    cameraView: "user",
  }),
  components: {
    ProductCardVue,
  },
  created() {
    document.title = "GenAI Product Finder | Vrijraj Singh";
    this.startCamera();
  },
  methods: {
    startCamera() {
      navigator.mediaDevices
        .getUserMedia({
          video: {
            facingMode: this.cameraView,
            width: 1920,
            height: 1020,
          },
        })
        .then(function (stream) {
          var video = document.getElementById("videoElement");
          video.srcObject = stream;
        })
        .catch(function (err) {
          console.log("An error occurred: " + err);
        });
    },
    changeCamera() {
      this.cameraView = this.cameraView === "user" ? "environment" : "user";
      this.startCamera();
    },
    async getProductImage(productName) {
      const apiKey = this.searchKey;
      const cx = this.cxKey;
      const apiUrl = `https://www.googleapis.com/customsearch/v1?key=${apiKey}&cx=${cx}&searchType=image&q=${encodeURIComponent(
        productName
      )}`;

      try {
        const response = await fetch(apiUrl);
        const data = await response.json();

        if (data.items && data.items.length > 0) {
          const imageUrl = data.items[1].link;
          return imageUrl;
        } else {
          return "";
        }
      } catch (error) {
        console.error("Error fetching product image:", error);
        return "";
      }
    },
    getURL(productName) {
      const baseUrl = "https://www.google.com/search";
      const queryParams = new URLSearchParams({
        tbm: "shop",
        q: productName.replace(/\s+/g, "+"), // Replace spaces with '+'
      });
      return `${baseUrl}?${queryParams}`;
    },
    async getData() {
      this.loading = true;
      this.finalData = []
      this.res = []
      var video = document.getElementById("videoElement");
      var canvas = document.getElementById("canvas");
      var context = canvas.getContext("2d");
      // Draw the current frame onto the canvas
      context.drawImage(video, 0, 0, canvas.width, canvas.height);

      // Convert the canvas content to a data URL representing the captured image
      var dataURL = canvas
        .toDataURL("image/png")
        .split("data:image/png;base64,")[1];
      // let url = "https://genai-backend-production.up.railway.app/api/v1/trip/test";
      // let url ="http://127.0.0.1:5001/vrij-web/us-central1/helloWorld"
      // let url = "http://localhost:3000/api/v1/trip/test";
      // const data = { image: dataURL };
      const MODEL_NAME = "gemini-1.5-pro-latest";
      const API_KEY = "AIzaSyCKmk8zCAY_pcjqI_haNsm3yPqiJqvu21I";
      const genAI = new GoogleGenerativeAI(API_KEY);
      const model = genAI.getGenerativeModel({ model: MODEL_NAME });
      const generationConfig = {
        temperature: 0.9,
        topK: 1,
        topP: 1,
        maxOutputTokens: 30720,
      };
      const safetySettings = [
        {
          category: HarmCategory.HARM_CATEGORY_HARASSMENT,
          threshold: HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
        },
        {
          category: HarmCategory.HARM_CATEGORY_HATE_SPEECH,
          threshold: HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
        },
        {
          category: HarmCategory.HARM_CATEGORY_SEXUALLY_EXPLICIT,
          threshold: HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
        },
        {
          category: HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT,
          threshold: HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
        },
      ];
      const parts = [
        {
          inlineData: {
            mimeType: "image/jpeg",
            data: dataURL,
          },
        },
        {
          text: `
            Identify multiple objects in the image and provide information about each object, including its name, company, description, and 
            similar products based on product details and
            ignore the object such as human, man, woman and child.

            Example:
            Given the image, identify the products and their respective companies. For each product detected:
            
            1. Name of the product:
            2. Product description (in 100 words max):
            3. 4 Similar products (if any):
            
            Return the data in the following JSON template:
            
            [
                {
                    "name": "Name of the product",
                    "description": "This product is used for ABC and its use cases are XYZ.",
                    "similar_products": ["Similar Product 1", "Similar Product 2", "Similar Product 3", "Similar Product 4", ...]
                },
                {
                    "product_name": "Name of another product",
                    "description": "This product is used for DEF and its use cases are UVW.",
                    "similar_products": ["Similar Product A", "Similar Product B", "Similar Product C", "Similar Product D", ...]
                },
                ...
            ]

            remove ... from similar_products from the results and remove json word
            
            `,
        },
      ];

      try {
        this.loading = true;
        const result = await model.generateContent({
          contents: [{ role: "user", parts }],
          generationConfig,
          safetySettings,
        });
        const response = result.response;
        // console.log('response', response.text());
        this.res = await JSON.parse(response.text().replace(/\n/g, ""));
        // console.log("this.res", this.res);

        this.finalData = await this.res.map(async (p) => {
          p.image = await this.getProductImage(p.name);
          p.url = await this.getURL(p.name);
          p.similar_products = await Promise.all(
            p.similar_products.map(async (sp) => {
              return {
                name: sp,
                url: await this.getURL(sp),
                img: await this.getProductImage(sp),
              };
            })
          );
          return p;
        });
        this.finalData = await Promise.all(this.finalData);
        this.loading = false;
      } catch (error) {
        console.log(error);
        this.loading = false;
      }
    },
  },
};
</script>

