<template>
  <section class="section get-started">
    <div class="container">
      <p
        id="get-started"
        class="title is-1"
      >
        开始
      </p>
      <p class="subtitle is-3">
        清理仓库从来没有这么快或这么容易！
      </p>

      <div class="columns">
        <div class="column is-half">
          <div class="step">
            <h3 class="step__title">
              <div class="step__num">
                <div class="step__num__content">
                  1
                </div>
              </div>
              从GitHub获取个人访问令牌
            </h3>

            <div class="content">
              <ol>
                <li>单击下面的按钮，在新窗口中访问GitHub.com。</li>
                <li>
                  滚动到底部并单击 <b>生成令牌</b>.
                </li>
                <li>复制生成的令牌并将其粘贴到下面。</li>
              </ol>
            </div>

            <a
              href="https://github.com/settings/tokens/new?scopes=delete_repo,repo&amp;description=Repo%20Remover%20Token"
              class="button is-link is-outlined"
              target="_blank"
            >
              <b-icon
                icon="chevron-right"
                size="is-small"
              />
              <span>
                获取令牌
              </span>
            </a>
          </div>

          <div class="step">
            <h3 class="step__title">
              <div class="step__num">
                <div class="step__num__content">
                  2
                </div>
              </div>
              选择存储库
            </h3>
            <b-field
              label="请输入您的个人访问令牌"
              :message="tokenInputMessage"
              :type="tokenInputType"
            >
              <b-input
                v-model="$root.$data.token"
                required
                placeholder="个人访问令牌"
                minlength="40"
                maxlength="40"
                autocomplete="false"
                pattern="[a-zA-Z0-9]+"
                :has-counter="false"
                :loading="tokenIsLoading"
                @input.native="onTokenInput"
              >
                />
              </b-input>
            </b-field>
            <small>
              <span class="tag is-info">
                放心使用：
              </span> 我们不会上传或不会保存你的令牌，但为了安全起见，您应该在使用后删除令牌。
            </small>
          </div>

          <b-button
            :disabled="!(hasToken && hasValidToken)"
            size="is-medium"
            type="is-primary"
            @click="onSubmit"
          >
            确定
          </b-button>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  data() {
    return {
      tokenInputType: "",
      tokenInputMessage: "",
      tokenIsLoading: false,
      hasValidToken: false,
      hasTokenError: false
    };
  },
  computed: {
    hasToken() {
      return this.$root.$data.token.length === 40;
    }
  },
  watch: {
    "$root.$data.token"(newValue) {
      if (newValue && this.hasToken) {
        this.$apollo.queries.gitHubViewer.options.context.headers.authorization = `Bearer ${this.$root.$data.token}`;
        this.$apollo.queries.gitHubViewer.start();
      }
    }
  },
  mounted() {
    this.$apollo.queries.gitHubViewer.setOptions({
      fetchPolicy: "no-cache"
    });
  },
  methods: {
    onSubmit(evt) {
      evt.preventDefault();
      this.$router.push("details");
    },
    onTokenInput(evt) {
      const target = evt.target;
      const validity = target.validity;
      target.setCustomValidity("");

      this.tokenInputType = "";
      this.tokenInputMessage = "";
      this.hasValidToken = false;
      this.$apollo.queries.gitHubViewer.stop();
      this.tokenIsLoading = false;

      if (!validity.valid) {
        if (validity.tooShort || validity.tooLong) {
          target.setCustomValidity(
            "个人访问令牌的长度必须恰好为40个字符"
          );
        }

        if (validity.patternMismatch) {
          target.setCustomValidity(
            "个人访问令牌应仅包括字母和数字。"
          );
        }
      } else {
        if (this.hasToken) this.tokenIsLoading = true;
      }
    }
  },
  apollo: {
    gitHubViewer() {
      return {
        context: {
          headers: {
            "User-Agent": "wuyuan"
          }
        },
        query: require("@/graphql/GitHubViewer.gql"),
        update: data => data.viewer,
        result({ data, loading, networkStatus }) {
          if (data && data.viewer && data.viewer.login) {
            this.hasValidToken = true;
            this.tokenInputType = "is-success";
            this.tokenIsLoading = false;
            this.$root.$data.login = data.viewer.login;
            this.tokenInputMessage = "成功！此令牌有效";
          }
        },
        // Error handling
        error(error) {
          this.hasTokenError = true;
          this.tokenInputType = "is-danger";
          this.tokenInputMessage =
            "错误：此令牌似乎无效。请验证令牌是否正确";
        },
        skip() {
          return true;
        }
      };
    }
  }
};
</script>

<style lang="scss">
.get-started {
  padding: 4em 1em;

  h3.title {
    align-items: center;
  }

  .subtitle {
    margin-bottom: 4rem !important;
  }

  .notification {
    margin-top: 1em;
  }
}
</style>
