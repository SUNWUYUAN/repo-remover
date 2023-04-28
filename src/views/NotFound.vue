<template>
  <main>
    <section class="padded">
      <div class="container">
        <h1 class="title is-1">
          404 - Page Not Found
        </h1>
        <h2 class="subtitle">
          我们似乎找不到您要查找的页面。
        </h2>

        <p>
          <router-link
            to="/"
            class="button is-link is-medium"
            type="submit"
          >
            <b-icon
              icon="chevron-circle-left"
              size="is-small"
            />
            <span>返回</span>
          </router-link>
        </p>
      </div>
    </section>
  </main>
</template>

<script>
export default {
  name: "NotFound"
};
</script>
