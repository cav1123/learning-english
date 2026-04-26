<template>
  <div class="min-h-screen pb-10 bg-white">
    <div
      v-if="isDrawerOpen"
      @click="toggleDrawer"
      class="fixed inset-0 bg-black/40 z-40 transition-opacity"
    ></div>

    <aside
      :class="[
        'fixed top-0 left-0 bottom-0 w-[85%] max-w-sm bg-white z-50 shadow-2xl flex flex-col transition-transform duration-300',
        isDrawerOpen ? 'translate-x-0' : '-translate-x-full',
      ]"
    >
      <div
        class="p-5 border-b border-gray-100 flex justify-between items-center bg-white"
      >
        <h2 class="font-bold text-gray-900">カテゴリ選択</h2>
        <button @click="toggleDrawer" class="text-gray-400 p-1">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-6 w-6"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
        </button>
      </div>

      <nav class="flex-1 overflow-y-auto p-4 space-y-6 hide-scrollbar">
        <div class="mb-8">
          <button
            @click="selectBookmarkMode"
            :class="[
              'w-full flex items-center justify-between text-[11px] font-bold uppercase tracking-widest p-3 rounded-lg border transition-all',
              isOnlyBookmarksMode
                ? 'bg-blue-600 border-blue-600 text-white shadow-sm'
                : 'bg-white border-gray-200 text-gray-500 hover:border-gray-400',
            ]"
          >
            <span>復習モード (チェック済み)</span>
            <span
              :class="[
                'px-2 py-0.5 rounded-full text-[10px]',
                isOnlyBookmarksMode ? 'bg-white/20' : 'bg-gray-100',
              ]"
            >
              {{ bookmarkCount }}
            </span>
          </button>
        </div>

        <div
          v-for="(chapters, bookName) in vocabularyData"
          :key="bookName"
          class="mb-6"
        >
          <button
            @click="selectCategory([bookName])"
            :class="[
              'w-full flex justify-between items-center text-[10px] font-black uppercase tracking-widest mb-3 px-2 py-1.5 rounded transition-colors',
              isActive([bookName])
                ? 'bg-gray-800 text-white'
                : 'text-gray-400 hover:bg-gray-50',
            ]"
          >
            <span>{{ bookName }}</span>
            <span class="opacity-80"
              >{{ getCheckedCount(chapters) }} /
              {{ getTotalCount(chapters) }}</span
            >
          </button>

          <div class="space-y-4 pl-3 ml-1 border-l border-gray-100">
            <div v-for="(sections, chapterName) in chapters" :key="chapterName">
              <button
                @click="selectCategory([bookName, chapterName])"
                :class="[
                  'w-full flex justify-between items-center text-left text-xs font-bold mb-2 p-1 rounded transition-colors',
                  isActive([bookName, chapterName])
                    ? 'text-blue-600 bg-blue-50/50'
                    : 'text-gray-700 hover:text-blue-500',
                ]"
              >
                <span>{{ chapterName }}</span>
                <span
                  :class="[
                    'text-[9px] font-normal',
                    getCheckedCount(sections) > 0
                      ? 'text-red-500 font-bold'
                      : 'opacity-50',
                  ]"
                >
                  {{ getCheckedCount(sections) }} /
                  {{ getTotalCount(sections) }}
                </span>
              </button>

              <div
                v-if="!Array.isArray(sections)"
                class="flex flex-col gap-1 mt-1 pl-1"
              >
                <button
                  v-for="(words, sectionName) in sections"
                  :key="sectionName"
                  @click="selectCategory([bookName, chapterName, sectionName])"
                  :class="[
                    'w-full flex justify-between items-center text-left text-[11px] px-2 py-1.5 rounded-md transition-all',
                    isActive([bookName, chapterName, sectionName])
                      ? 'bg-blue-600 text-white font-bold'
                      : 'text-gray-500 hover:bg-gray-50',
                  ]"
                >
                  <span>{{ sectionName }}</span>
                  <span
                    :class="[
                      'text-[9px] font-normal',
                      !isActive([bookName, chapterName, sectionName]) &&
                      getCheckedCount(words) > 0
                        ? 'text-red-400 font-bold'
                        : 'opacity-60',
                    ]"
                  >
                    {{ getCheckedCount(words) }} / {{ getTotalCount(words) }}
                  </span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </nav>
    </aside>

    <header class="bg-slate-50 border-b border-slate-200 sticky top-0 z-20">
      <div
        class="max-w-2xl mx-auto px-4 py-2.5 flex items-center justify-between"
      >
        <button
          @click="toggleDrawer"
          class="p-2 -ml-2 text-gray-600 hover:text-gray-900"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-5 w-5"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M4 6h16M4 12h16m-7 6h7"
            />
          </svg>
        </button>
        <div class="text-center flex-1 px-2">
          <h1 class="text-sm font-bold text-gray-800 tracking-tight">
            LearningEnglish
          </h1>
          <div
            class="text-[10px] text-gray-500 font-medium mt-0.5 truncate uppercase tracking-wider"
          >
            {{ isOnlyBookmarksMode ? "復習リスト" : currentPath.join(" / ") }}
          </div>
        </div>
        <div class="flex items-center gap-2">
          <button
            @click="toggleAllWords"
            :class="[
              'p-2 transition-colors',
              isAllRevealed
                ? 'text-blue-600'
                : 'text-gray-400 hover:text-blue-600',
            ]"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-5 w-5"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                v-if="!isAllRevealed"
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M15 12a3 3 0 11-6 0 3 3 0 016 0zM2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"
              />
              <path
                v-else
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.88 9.88l-3.29-3.29m7.532 7.532l3.29 3.29M3 3l18 18"
              />
            </svg>
          </button>
          <div class="w-10 flex justify-end">
            <span
              class="bg-gray-200 px-1.5 py-0.5 rounded text-[9px] font-bold text-gray-600"
              >{{ displayWords.length }}</span
            >
          </div>
        </div>
      </div>
    </header>

    <main class="max-w-2xl mx-auto px-3 mt-4 space-y-2">
      <div
        v-if="displayWords.length === 0"
        class="text-center py-20 text-gray-400 text-xs"
      >
        表示対象がありません
      </div>

      <div
        v-for="(item, index) in displayWords"
        :key="item.id"
        class="bg-white px-4 py-3 rounded-lg border border-gray-100 flex flex-col gap-1.5 cursor-pointer relative hover:border-blue-200 shadow-sm transition-all active:scale-[0.98]"
        @click="toggleWord(item.id)"
      >
        <div class="flex justify-between items-start">
          <div class="flex flex-col">
            <span
              class="text-[9px] font-bold text-gray-300 uppercase tracking-tighter"
              >#{{ index + 1 }}</span
            >
            <span
              v-if="isOnlyBookmarksMode && bookmarks[item.id]"
              class="text-[8px] text-red-400 font-medium italic"
            >
              Checked: {{ bookmarks[item.id].date }}
            </span>
          </div>
          <div class="flex items-center">
            <button
              @click.stop="toggleBookmark(item.id)"
              :class="[
                'p-1.5 transition-colors',
                bookmarks[item.id]
                  ? 'text-red-500'
                  : 'text-gray-200 hover:text-gray-400',
              ]"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4"
                :fill="bookmarks[item.id] ? 'currentColor' : 'none'"
                viewBox="0 0 24 24"
                stroke="currentColor"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M5 5a2 2 0 012-2h10a2 2 0 012 2v16l-7-3.5L5 21V5z"
                />
              </svg>
            </button>
            <button
              @click.stop="speak(item.phrase)"
              class="p-1.5 text-gray-200 hover:text-blue-500"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4"
                viewBox="0 0 20 20"
                fill="currentColor"
              >
                <path
                  fill-rule="evenodd"
                  d="M9.383 3.076A1 1 0 0110 4v12a1 1 0 01-1.707.707L4.586 13H2a1 1 0 01-1-1V8a1 1 0 011-1h2.586l3.707-3.707a1 1 0 011.09-.217zM14.657 14.657a1 1 0 01-1.414-1.414A6.732 6.732 0 0015 8.5c0-1.815-.718-3.463-1.884-4.66a1 1 0 011.439-1.388A8.73 8.73 0 0117 8.5c0 2.373-.939 4.527-2.457 6.157z"
                  clip-rule="evenodd"
                />
              </svg>
            </button>
          </div>
        </div>

        <div class="flex flex-col gap-0.5">
          <div
            class="text-[13px] text-gray-500 font-medium"
            v-html="item.translation"
          ></div>
          <div class="text-base text-gray-900 font-bold tracking-tight">
            <span v-html="formatPhrase(item.phrase, item.word, item.id)"></span>
          </div>
        </div>
        <!-- 回答 -->
        <Transition name="fade">
          <div
            v-if="isAllRevealed || revealedIds.has(item.id)"
            class="text-xs pt-1"
          >
            <span
              class="text-[9px] font-bold text-blue-600 bg-blue-50 px-1 rounded mr-1.5 uppercase"
              >{{ item.pos }}</span
            >
            <span class="text-gray-600">{{ item.meaning }}</span>
          </div>
        </Transition>
      </div>
    </main>
  </div>
</template>

<script>
// データのインポート
import { vocabularyData } from "./data/vocabularyData.js";
export default {
  data() {
    return {
      isDrawerOpen: false,
      isAllRevealed: false,
      isOnlyBookmarksMode: false,
      currentPath: [
        "金のフレーズ",
        "Short Break",
        "TOEICに出る注意すべき日常単語",
      ],
      revealedIds: new Set(),
      bookmarks: JSON.parse(localStorage.getItem("le_bookmarks_v2") || "{}"),
      vocabularyData: vocabularyData,
    };
  },
  computed: {
    displayWords() {
      if (this.isOnlyBookmarksMode) {
        const all = this.collectAllWords(this.vocabularyData);
        return all.filter((w) => this.bookmarks[w.id]);
      }
      let current = this.vocabularyData;
      for (const key of this.currentPath) {
        if (current[key]) current = current[key];
        else return [];
      }
      return this.collectAllWords(current);
    },
    bookmarkCount() {
      return Object.keys(this.bookmarks).length;
    },
  },
  methods: {
    toggleDrawer() {
      this.isDrawerOpen = !this.isDrawerOpen;
    },
    toggleWord(id) {
      if (this.revealedIds.has(id)) {
        this.revealedIds.delete(id);
      } else {
        this.revealedIds.add(id);
        const item = this.displayWords.find((w) => w.id === id);
        if (item) this.speak(item.phrase);
      }
    },
    toggleAllWords() {
      this.isAllRevealed = !this.isAllRevealed;
      if (!this.isAllRevealed) this.revealedIds.clear();
    },
    toggleBookmark(wordId) {
      if (this.bookmarks[wordId]) {
        delete this.bookmarks[wordId];
      } else {
        const now = new Date();
        this.bookmarks[wordId] = {
          date: `${now.getFullYear()}/${String(now.getMonth() + 1).padStart(2, "0")}/${String(now.getDate()).padStart(2, "0")}`,
        };
      }
      localStorage.setItem("le_bookmarks_v2", JSON.stringify(this.bookmarks));
      // オブジェクトの変更を検知させるため再代入
      this.bookmarks = { ...this.bookmarks };
    },
    selectCategory(path) {
      this.isOnlyBookmarksMode = false;
      this.currentPath = path;
      this.isAllRevealed = false;
      this.revealedIds.clear();
      this.isDrawerOpen = false;
    },
    selectBookmarkMode() {
      this.isOnlyBookmarksMode = true;
      this.isAllRevealed = false;
      this.revealedIds.clear();
      this.isDrawerOpen = false;
    },
    collectAllWords(obj) {
      if (!obj) return [];
      if (Array.isArray(obj)) return obj;
      let results = [];
      for (let key in obj) {
        results = results.concat(this.collectAllWords(obj[key]));
      }
      return results;
    },
    getCheckedCount(node) {
      const words = this.collectAllWords(node);
      return words.filter((w) => this.bookmarks[w.id]).length;
    },
    getTotalCount(node) {
      return this.collectAllWords(node).length;
    },
    isActive(path) {
      if (this.isOnlyBookmarksMode) return false;
      return path.every((val, index) => this.currentPath[index] === val);
    },
    formatPhrase(phrase, word, id) {
      const regex = new RegExp(
        word.replace(/[.*+?^${}()|[\]\\]/g, "\\$&"),
        "gi",
      );
      const isRevealed = this.isAllRevealed || this.revealedIds.has(id);

      return phrase.replace(regex, (matched) => {
        const baseClass =
          "px-1 rounded transition-all duration-200 cursor-pointer ";
        const stateClass = isRevealed
          ? "bg-transparent text-blue-600 font-bold"
          : "bg-slate-200 text-transparent border-b-2 border-slate-400 select-none";
        return `<span class="${baseClass} ${stateClass}">${matched}</span>`;
      });
    },
    speak(text) {
      if (!text) return;
      window.speechSynthesis.cancel();
      const uttr = new SpeechSynthesisUtterance(text);
      uttr.lang = "en-US";
      uttr.rate = 0.9;
      window.speechSynthesis.speak(uttr);
    },
  },
};
</script>

<style scoped>
/* フェードインのアニメーション設定 */
.fade-enter-active,
.fade-leave-active {
  transition:
    opacity 0.3s ease,
    transform 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(
    -5px
  ); /* 少し上から降りてくる感じにするとバイブス上がります */
}
.hide-scrollbar::-webkit-scrollbar {
  display: none;
}
</style>
