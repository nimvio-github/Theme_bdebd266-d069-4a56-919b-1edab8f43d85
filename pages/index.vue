<template>
  <NuxtLayout
    :name="data.Data.layout[0].Data.layoutName"
    :placeholders="data.Data.layout[0].Data.placeholders"
  >
    <template #main>
      <!-- content for the main slot -->
      <component-renderer
        v-if="data.Data.widgetContent"
        :components="data.Data.widgetContent"
      ></component-renderer>
    </template>
  </NuxtLayout>
</template>

<script setup>
import { getContentById } from "~~/utils/dataFetching";
import getFullSlug from "~~/composables/getFullSlug";

// Get the content from Nimvio via ContentID instead of pageSlug
const { contentId } = getFullSlug();

const { data } = await useAsyncData(contentId, async ({ $gqlClient }) => {
  const { data: response } = await getContentById($gqlClient, contentId, {
    deep: true,
  });

  return response;
});

const updateContentById = (content, id, newContent, cache = {}) => {
  if (cache[content.ContentID]) return null;
  cache[content.ContentID] = true;
  if (content.ContentID === id) {
    content.Data = newContent;
    return content;
  }
  const componentDataKeys = Object.keys(content.Data);
  for (let i = 0; i < componentDataKeys.length; i++) {
    const componentDataKey = componentDataKeys[i];
    if (Array.isArray(content.Data[componentDataKey])) {
      for (let j = 0; j < content.Data[componentDataKey].length; j++) {
        const found = updateContentById(
          content.Data[componentDataKey][j],
          id,
          newContent,
          cache
        );
        if (found) {
          content.Data[componentDataKey][j] = found;
          return content;
        }
      }
    }
  }
  return null;
};

const { $nimvioSdk } = useNuxtApp();
onBeforeMount(() => {
  $nimvioSdk.livePreviewUtils.onPreviewContentChange((formData) => {
    const newContent = updateContentById(
      data.value,
      formData.id,
      formData.formData
    );
    if (newContent) {
      data.value = newContent;
    }
  });
});

useHead({
  title: data.value?.Data.pageTitle,
});
</script>

<script>
export default {
  name: "Home",
};
</script>
