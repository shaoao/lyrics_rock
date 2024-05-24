<template>
  <div class="p-4">
    <h2 class="text-2xl font-bold mb-4">文件列表</h2>
    <div class="bg-white shadow rounded-lg p-6 mx-auto w-70">
      <label class="input input-bordered flex items-center gap-2">
        <input
          type="text"
          v-model="searchTerm"
          class="grow"
          placeholder="SearchName"
        />
        <svg
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 16 16"
          fill="currentColor"
          class="w-4 h-4 opacity-70"
        >
          <path
            fill-rule="evenodd"
            d="M9.965 11.026a5 5 0 1 1 1.06-1.06l2.755 2.754a.75.75 0 1 1-1.06 1.06l-2.755-2.754ZM10.5 7a3.5 3.5 0 1 1-7 0 3.5 3.5 0 0 1 7 0Z"
            clip-rule="evenodd"
          />
        </svg>
      </label>
      <table class="table w-full table-zebra">
        <thead>
          <tr>
            <th class="text-xl w-1/3">标题</th>
            <th class="text-xl w-1/3">格式</th>
            <th class="text-xl w-1/3">操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="file in paginatedFiles" :key="file.title">
            <td>{{ file.title }}</td>
            <td>{{ file.format }}</td>
            <td>
              <button
                class="btn btn-secondary btn-sm mr-2"
                @click="previewFile(file)"
              >
                预览
              </button>
              <a
                :href="file.url"
                download
                :class="[
                  'btn btn-primary btn-sm',
                  { 'btn-disabled': !file.url },
                ]"
                >下载</a
              >
            </td>
          </tr>
        </tbody>
      </table>

      <!-- 分页组件 -->
      <br />
      <div class="join grid grid-cols-2 mt-4">
        <button
          class="join-item btn btn-outline"
          @click="currentPage > 1 ? currentPage-- : null"
          :disabled="currentPage === 1"
        >
          Previous
        </button>
        <button
          class="join-item btn btn-outline"
          @click="currentPage < totalPages ? currentPage++ : null"
          :disabled="currentPage === totalPages"
        >
          Next
        </button>
      </div>
    </div>

    <!-- 预览 -->
    <div
      v-if="selectedFileContent"
      class="modal modal-open glass sm:modal-middle"
      @click="closeModal"
    >
      <div class="modal-box" @click.stop>
        <form method="dialog">
          <button
            class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
            @click="selectedFileContent = null"
          >
            ✕
          </button>
        </form>
        <h3 class="text-lg font-bold">{{ selectedFile?.title }}</h3>
        <br />
        <pre class="bg-gray-100 p-4 rounded mx-auto w-75">
          {{ selectedFileContent }}
        </pre>
        <div class="modal-action">
          <button
            class="btn btn-neutral btn-sm mr-2"
            @click="copyTextToClipboard(selectedFileContent)"
          >
            复制
          </button>
          <button class="btn btn-sm mr-2" @click="selectedFileContent = null">
            关闭
          </button>
        </div>
      </div>
      <!-- 复制成功 -->
      <div v-show="showAlert" role="alert" class="alert alert-success w-80">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="stroke-current shrink-0 h-6 w-6"
          fill="none"
          viewBox="0 0 24 24"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"
          />
        </svg>
        <span>Copy Success!</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';

interface File {
  title: string;
  album?: string;
  format: string;
  url: string;
}

export default defineComponent({
  setup() {
    const files = ref<File[]>([
      { title: "人洞山", album: "瓦合", format: "SRT", url: "./Resources/人洞山.srt" },
      { title: "如常", album: "如常", format: "SRT", url: "./Resources/如常.srt" },
      { title: "闹海", album: "它在时间门外", format: "SRT", url: "./Resources/闹海.srt" },
      { title: "人生逃避号", format: "SRT", url: "./Resources/人生逃避号.srt" },
      { title: "我们", album: "丑奴儿", format: "SRT", url: "./Resources/我们.srt" },
      { title: "在", album: "丑奴儿", format: "SRT", url: "./Resources/在.srt" },
      { title: "鬼", album: "丑奴儿", format: "SRT", url: "./Resources/鬼.srt" },
      { title: "缸", album: "瓦合", format: "SRT", url: "./Resources/缸.srt" },
      { title: "八", album: "瓦合", format: "SRT", url: "./Resources/八.srt" },
      { title: "床", album: "瓦合", format: "SRT", url: "./Resources/床.srt" },
      { title: "但", album: "瓦合", format: "SRT", url: "./Resources/但.srt" },
      { title: "烂泥", album: "丑奴儿", format: "SRT", url: "./Resources/烂泥.srt" },
      { title: "空", album: "瓦合", format: "SRT", url: "./Resources/空.srt" },
      { title: "老张", album: "瓦合", format: "SRT", url: "./Resources/老张.srt" },
      { title: "情歌", album: "丑奴儿", format: "SRT", url: "./Resources/情歌.srt" },
      { title: "勇敢的人", album: "丑奴儿", format: "SRT", url: "./Resources/勇敢的人.srt" },
      { title: "丑", album: "丑奴儿", format: "SRT", url: "./Resources/丑.srt" },
      { title: "顶楼", album: "未发行", format: "SRT", url: "./Resources/顶楼.srt" },
      { title: "等", album: "丑奴儿", format: "SRT", url: "./Resources/等.srt" },
      { title: "山海", album: "丑奴儿", format: "SRT", url: "./Resources/山海.srt" },
    ]);

    const searchTerm = ref("");
    const currentPage = ref(1);
    const itemsPerPage = ref(10);
    const selectedFile = ref<File | null>(null);
    const selectedFileContent = ref<string | null>(null);
    const showAlert = ref(false);

    const filteredFiles = computed(() => {
      const term = searchTerm.value.trim().toLowerCase();
      return files.value.filter(file =>
        file.title.toLowerCase().includes(term)
      );
    });

    const paginatedFiles = computed(() => {
      const startIndex = (currentPage.value - 1) * itemsPerPage.value;
      const endIndex = startIndex + itemsPerPage.value;
      return filteredFiles.value.slice(startIndex, endIndex);
    });

    const totalPages = computed(() => {
      return Math.ceil(filteredFiles.value.length / itemsPerPage.value);
    });

    const previewFile = async (file: File) => {
      selectedFile.value = file;
      try {
        const response = await fetch(file.url);
        if (response.ok) {
          selectedFileContent.value = await response.text();
        } else {
          selectedFileContent.value = "无法加载文件内容。";
        }
      } catch (error) {
        selectedFileContent.value = "加载文件时出错。";
      }
    };

    const closeModal = () => {
      selectedFileContent.value = null;
    };

    const copyTextToClipboard = (text: string) => {
      const textarea = document.createElement("textarea");
      textarea.value = text;
      document.body.appendChild(textarea);
      textarea.select();
      const successful = document.execCommand("copy");
      document.body.removeChild(textarea);

      if (successful) {
        console.log("文本已成功复制到剪贴板");
        showConfirmationAlert();
      } else {
        console.error("复制到剪贴板失败");
      }
    };

    const showConfirmationAlert = () => {
      showAlert.value = true;
      setTimeout(() => {
        showAlert.value = false;
      }, 3000);
    };

    return {
      files,
      searchTerm,
      currentPage,
      itemsPerPage,
      selectedFile,
      selectedFileContent,
      showAlert,
      filteredFiles,
      paginatedFiles,
      totalPages,
      previewFile,
      closeModal,
      copyTextToClipboard,
      showConfirmationAlert,
    };
  },
});
</script>

<style scoped>
.container {
  max-width: 800px;
}
</style>
