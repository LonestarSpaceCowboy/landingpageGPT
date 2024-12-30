<template>
  <v-card>
    <v-card-title>Welcome!</v-card-title>
    <v-card-text>Here's a great offer for you!</v-card-text>
    <v-divider />
    <v-progress-linear
      indeterminate
      v-if="!generatedOfferText || !parsedContent"
    ></v-progress-linear>
    <v-card-text>
      <div>
        <!-- Render the block of text with the clickable link -->
        <div v-html="parsedContent"></div>
      </div>
    </v-card-text>
    <v-card-actions>
      <v-btn append-icon="mdi-refresh" @click="regenHandler"
        >Regenerate</v-btn
      >
    </v-card-actions>
  </v-card>
</template>

<script>
import { OpenAI } from "openai";
import.meta.env;

export default {
  props: ["userData", "productData"],
  data() {
    return {
      generatedOfferText: "",
      parsedContent: "",
    };
  },
  mounted() {
    this.generateAdCopy();
  },
  computed: {
    defaultPrompt() {
      return `Act as a marketing genius with deep expertise in persuasive, emotionally compelling copywriting. 
Your task is to create a short-form ad copy (300 words max) designed to persuade a specific user to engage with an offer.

[USER INFORMATION]
- Name: ${this.userData.name}
- Recent or Favorite Websites: ${this.userData.recentSites.join(", ")}
- Additional Behaviors or Interests: [Use websites to determine]
- Location: ${this.userData.location}

[OFFER INFORMATION]
              -Offer Information:
              -Product Type: ${this.productData.type}
              -Product Name: ${this.productData.productName}
              -Company: ${this.productData.company}
              -Offer: ${this.productData.offer}
              -Offer URL: ${this.productData.offerURL}

[INSTRUCTIONS]
1. **Infer the customer avatar**: Based on the user’s websites, behaviors, and/or location, deduce their potential interests, goals, or concerns.
2. **Speak directly to this avatar**: Use empathetic, benefit-focused language that acknowledges why they might need or desire the product/service.
3. **Highlight 2–3 key benefits** relevant to the user’s specific pain points or lifestyle. Use bullet points and include short emotional hooks or emojis if appropriate.
4. **Keep the copy under 300 words** for clarity and impact.
5. **End with a clear, compelling call-to-action** that references the Offer URL (e.g., "Click here to [Benefit].").

Use warm, inspiring language that instills trust and excitement, positioning the offer as the perfect solution. 
Your goal is to make the user feel the product/service was practically made for them.`;

      //   `Act as a marketing genius that is an expert at selling any product. Below is some known information about an individual visiting our website.
      //         Use the information to formulate a compelling ad copy to prompt the user to click on the offer link and sign up. Use emotional selling to draw a direct connection
      //         as to how the offer would benefit them.  Provide two to three bullet points as to what benefits someone like them are likely to gain from the offer, based on their problems, consumption habits and interests.

      //         Always keep the link at the end and always have the link text be a clear call to action like "click here to...".
      //         Keep the offer shorter than 100 words.  Use formatting like bullet points, emojis and new lines to keep interest.

      //         Known User Information:
      //         Name: ${this.userData.name}
      //         Recently Visited Websites: ${this.userData.recentSites.join(", ")}
      //         Location: ${this.userData.location}

      //   Offer Information:
      //   Product Type: ${this.productData.type}
      //   Product Name: ${this.productData.productName}
      //   Company: ${this.productData.company}
      //   Offer: ${this.productData.offer}
      //   Offer URL: ${this.productData.offerURL}
      //         `;
    },
  },
  methods: {
    async generateAdCopy() {
      const openai = new OpenAI({
        apiKey: import.meta.env.VITE_OPENAI_API_KEY,
        dangerouslyAllowBrowser: true,
      });
      const prompt = this.defaultPrompt;
      const completion = await openai.chat.completions.create({
        model: "gpt-4o",
        messages: [
          { role: "system", content: "You are a helpful assistant." },
          {
            role: "user",
            content: prompt,
          },
        ],
      });

      this.generatedOfferText = completion.choices[0].message.content;
      //format the offer link
      this.parseMarkdownLinks();
    },
    parseMarkdownLinks() {
      // Regex to match Markdown-style links
      const markdownLinkRegex = /\[(.*?)\]\((.*?)\)/g;

      // Replace Markdown links with HTML anchor tags
      this.parsedContent = this.generatedOfferText.replace(
        markdownLinkRegex,
        (match, label, url) => {
          return `<a href="${url}" target="_blank" rel="noopener noreferrer">${label}</a>`;
        }
      );
    },
    regenHandler() {
        this.parsedContent = '';
        this.generateAdCopy();
    }
  },
};
</script>
