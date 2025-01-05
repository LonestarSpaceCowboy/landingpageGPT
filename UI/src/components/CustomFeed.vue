<template>
  <v-card>
    <v-card-title>Welcome!</v-card-title>
    <v-card-text>Here's a great offer for you!</v-card-text>
    <v-divider />
    <v-progress-linear
      indeterminate
      v-if="loading"
    ></v-progress-linear>
    <v-card-text v-html="parsedContent"></v-card-text>
    <v-card-actions>
      <v-btn append-icon="mdi-refresh" @click="generateAdCopy"
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
      loading: true,
      generatedOfferText: "",
      parsedContent: "",
    };
  },
  mounted() {
    //comment during development so it doesn't keep using tokens while I'm testing....
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
- Product Type: ${this.productData.type}
- Product Name: ${this.productData.productName}
- Company: ${this.productData.company}
- Offer: ${this.productData.offer}
- Offer URL: ${this.productData.offerURL}

[INSTRUCTIONS]
1. **Infer the customer avatar**: Based on the user’s websites, behaviors, and/or location, deduce their potential interests, goals, or concerns.
2. **Speak directly to this avatar**: Use empathetic, benefit-focused language that acknowledges why they might need or desire the product/service.
3. **Highlight 2–3 key benefits** relevant to the user’s specific pain points or lifestyle. Use bullet points and include short emotional hooks or emojis if appropriate.
4. **Keep the copy under 300 words** for clarity and impact.
5. **End with a clear, compelling call-to-action** that references the Offer URL (e.g., "Click here to [Benefit].").
6. **Use bolding with two asterisks** to emphasize key points or benefits and make sure to use new lines on bolded text.**
Use warm, inspiring language that instills trust and excitement, positioning the offer as the perfect solution. 
Your goal is to make the user feel the product/service was practically made for them.`;
    }
  },
  watch: {
    generatedOfferText(newVal) {
      this.parseMarkdownLinks();
    }
  },
  methods: {
    regenHandler() {
        this.parsedContent = '';
        this.generateAdCopy();
    },
    async generateAdCopy() {
      this.loading=true;

      const openai = new OpenAI({
        apiKey: import.meta.env.VITE_OPENAI_API_KEY,
        dangerouslyAllowBrowser: true,
      });
      const prompt = this.defaultPrompt;
      const completion = await openai.chat.completions.create({
        model: "gpt-4",
        messages: [{ role: "user", content: prompt }],
      });
      this.generatedOfferText = completion.choices[0].message.content;
      this.loading=false;
    },
    parseMarkdownLinks() {
      // Regex to match Markdown-style links
      const markdownLinkRegex = /\[(.*?)\]\((.*?)\)/g;

      // Replace Markdown links with HTML anchor tags
      let content = this.generatedOfferText.replace(
        markdownLinkRegex,
        (match, label, url) => {
          return `<a href="${url}" target="_blank" rel="noopener noreferrer">${label}</a>`;
        }
      );

      // Regex to match text inside ** **
      const boldTextRegex = /\*\*(.*?)\*\*/g;

      // Replace **text** with <b>text</b> and add bullet points and new lines
      content = content.replace(
        boldTextRegex,
        (match, text) => {
          return `<li><b>${text}</b></li>`;
        }
      );

      // Wrap the content in <ul> tags
      this.parsedContent = `<ul>${content}</ul>`;
    },
  },
};
</script>
