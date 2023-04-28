<template>
  <div
    id="confirmAction"
    class="modal-card"
  >
    <header class="modal-card-head">
      <p class="modal-card-title has-text-weight-bold">
        {{ modalTitle }}
      </p>
    </header>

    <section class="modal-card-body">
      <b-message
        v-if="showDelete"
        type="is-danger"
      >
        <strong>
          警告：
        </strong>
        此操作无法撤消。 这将永久删除 {{ repos.length | pluralize("仓库", "仓库", { noNumber: true }) }}, {{ repos.length | pluralize("维基", "wiki", { noNumber: true }) }}, 问题和评论，并删除所有合作者关联。
      </b-message>

      <p>
        你确认 <strong>{{ showDelete ? "删除" : "存档" }}</strong> 以下 {{ repos.length | pluralize("个仓库", "个仓库", { noSingleValue: true }) }}?
      </p>

      <div class="content confirm-action-list">
        <ul class="">
          <li
            v-for="repo in repos"
            :key="repo.id"
          >
            {{ repo.name }}
          </li>
        </ul>
      </div>

      <b-field label="请键入您的GitHub用户名进行确认：">
        <b-input
          v-model="confirmUsername"
          autofocus
        />
      </b-field>
    </section>

    <footer class="modal-card-foot">
      <div class="buttons">
        <button
          class="button"
          type="button"
          @click="$parent.close()"
        >
          取消
        </button>
        <button
          :class="['button',
                   showDelete ? 'is-danger' : 'is-warning']"
          :disabled="confirmUsername !== $root.$data.login"
          @click="modifyRepos"
        >
        我知道我在干什么,请{{ showDelete ? "删除" : "存档" }}{{ repos.length | pluralize("这些存储库", "此存储库", { noNumber: true }) }}
        </button>
      </div>
    </footer>
  </div>
</template>

<script>
import { filters } from "@/mixins.js";

const pSettle = require("p-settle");
const Octokit = require("@octokit/rest");

export default {
  mixins: [filters],
  props: {
    showDelete: {
      type: Boolean,
      default: true
    },
    repos: {
      type: Array,
      default: function() {
        return [];
      }
    }
  },
  data() {
    return {
      confirmUsername: ""
    };
  },
  computed: {
    modalTitle() {
      return "确认 " + (this.showDelete ? "删除" : "存档");
    },
    modalOkText() {
      return (this.showDelete ? "删除" : "存档") + " 仓库";
    }
  },
  mounted() {
    this.octokit = new Octokit({
      auth: `token ${this.$root.$data.token}`,
      userAgent: "reporemover"
    });

    this.octokit.hook.error("request", error => {
      throw error;
    });
  },
  methods: {
    async modifyRepos() {
      let promises = this.repos.map(async repo => {
        try {
          if (this.showDelete) {
            await this.octokit.repos.delete({
              owner: repo.owner.login,
              repo: repo.name
            });

            window.fathom && fathom.trackGoal(process.env.VUE_APP_FATHOM_REPO_DELETED, 0);
          } else {
            await this.octokit.repos.update({
              owner: repo.owner.login,
              repo: repo.name,
              name: repo.name,
              archived: true
            });

            window.fathom && fathom.trackGoal(process.env.VUE_APP_FATHOM_REPO_ARCHIVED, 0);
          }
          return repo;
        } catch (error) {
          return Promise.reject({ error, repo });
        }
      });

      const results = await pSettle(promises);
      this.$root.$emit("repos-updated", this.showDelete, results);
      this.$parent.close();
    }
  }
};
</script>

<style lang="scss" scoped>
.confirm-action-list {
  margin-top: 0.5em;
  max-height: 50vh;
  overflow-y: auto;
}

.modal-warning {
  margin-top: 2em;
}

.modal-card-foot {
  justify-content: flex-end;

  .button {
    @include mobile {
      width: 100%;
      height: auto;
      margin-right: 0;
      white-space: normal;
    }
  }
}
</style>

