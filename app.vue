<template>
  <div v-bind="webLinkProps">
    <span
      v-if="'preview' in route.query"
      :data-nimvio-content-id="publicConfig.styleContentId"
      data-nimvio-template-name="styles"
      >Website Styles</span
    >
    <NuxtPage />
  </div>
</template>

<script setup lang="ts">
import { getContentById } from "./utils/dataFetching";
import { StylesData, StylesTemplate } from "./models/templates.model";

const { public: publicConfig } = useRuntimeConfig();
const route = useRoute();
const router = useRouter();

const webLinkProps = {
  "data-nimvio-project-id": publicConfig.projectId,
  "data-kontent-language-codename": "default",
};

const { data } = await useAsyncData("app", async ({ $gqlClient }) => {
  const { data: response } = await getContentById<StylesTemplate>(
    $gqlClient,
    publicConfig.styleContentId,
    { deep: true }
  );
  const pageData = response.Data;

  return {
    pageData,
    styles: pageData.styles,
  };
});

const customStyles = computed(() => {
  const styles = data.value.styles.map((style) => {
    const styleObj = {
      type: "text/css",
      innerHTML: style.Data.internalCss
        .replace(/<[^>]+>/g, "")
        .replace(/&nbsp;/g, "")
        .replace(/&quot;/g, '"')
        .replace(/&gt;/g, ">"),
    };
    return styleObj;
  });
  return styles;
});

const customLinks = computed(() => {
  const links = [];
  data.value.styles.forEach((style) => {
    // Add style from externalCss
    if (style.Data.externalCss) {
      links.push({
        rel: "stylesheet",
        as: "style",
        href: style.Data.externalCss,
      });
    }
    const formattedFonts = style.Data.fonts.map((font) => {
      const fontObj = {
        rel: "stylesheet",
        as: "style",
        href: font.Data.linkUrl,
      };
      return fontObj;
    });
    links.push(...formattedFonts);
  });
  return links;
});

const { $nimvioSdk } = useNuxtApp();
onBeforeMount(() => {
  $nimvioSdk.livePreviewUtils.onPreviewContentChange<StylesData>((formData) => {
    if (formData.id === publicConfig.styleContentId) {
      data.value.styles = formData.formData.styles;
    }
  });

  // Open the content page when the content is opened inside the Nimvio
  $nimvioSdk.livePreviewUtils.onOpenPreviewContent((data) => {
    const availableRoutes = publicConfig.routes;
    const isRouteExist = availableRoutes.find(
      (availableRoute) => availableRoute.ContentID === data.id
    );
    if (data.templateName.startsWith("Page")) {
      if (isRouteExist) {
        const contentId = data.id;
        const { route: openedRoute } = publicConfig.routes.find(
          (route) => route.ContentID === contentId
        );
        const currentPath =
          route.path !== "/" && route.path.endsWith("/")
            ? route.path.slice(0, -1)
            : route.path;
        currentPath !== openedRoute && router.push(openedRoute);
      }
    }
  });

  $nimvioSdk.livePreviewUtils.onNewPreviewContent((data) => {
    console.log("onNewPreviewContent", data);
  });
});

// Use function to make it reactive
useHead(() => ({
  bodyAttrs: {
    class: "font-sans text-dark-gray",
  },
  style: [...customStyles.value],
  link: [...customLinks.value],
}));
</script>
