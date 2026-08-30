# AGENTS.md

## Slidev Rules

- In slides.md, HTML blocks cannot contain blank lines. Blank lines inside an HTML block cause the content to be rendered as raw text instead of HTML. Keep all HTML within a block contiguous with no empty lines.
- To show content that replaces on each click (e.g. a sequence of images), use `v-switch` with numbered templates, not `v-click` with ranges. Example:
  ```
  <v-switch>
    <template #0>first</template>
    <template #1>second</template>
    <template #2>third</template>
  </v-switch>
  ```
- When placing images from the user's filesystem, always view them first to verify which image is which before assigning filenames or ordering.