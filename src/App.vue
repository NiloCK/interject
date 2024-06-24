<template>
  <div class="audio-player">
    <div class="player-controls">
      <button @click="togglePlay">{{ isPlaying ? "Pause" : "Play" }}</button>
      <input
        type="range"
        min="0"
        :max="duration"
        v-model="currentTime"
        @input="seek"
      />
      <span>{{ formatTime(currentTime) }} / {{ formatTime(duration) }}</span>
    </div>
    <div class="timeline">
      <div class="heatmap">
        <div
          v-for="(count, index) in heatmapData"
          :key="index"
          class="heatmap-segment"
          :style="{
            height: `${(count / maxCommentCount) * 100}%`,
            backgroundColor: `rgba(255, 0, 0, ${count / maxCommentCount})`,
          }"
        ></div>
        <div
          class="comment-window"
          :style="{
            left: commentWindowPosition + '%',
            width: commentWindowWidth + '%',
          }"
        ></div>
        <input
          type="range"
          min="0"
          :max="duration"
          v-model="currentTime"
          @input="seek"
          class="seek-bar"
        />
      </div>
      <input
        type="range"
        min="0"
        :max="duration"
        v-model="currentTime"
        @input="seek"
        class="seek-bar"
      />
    </div>
    <div class="comment-display">
      <CommentItem
        v-for="comment in topLevelComments"
        :key="comment.id"
        :comment="comment"
        :all-comments="visibleComments"
        :is-replying-to="
          replyingToComment && replyingToComment.id === comment.id
        "
        :replying-to-comment="replyingToComment"
        @reply="replyToComment"
      />
    </div>
    <div class="comment-input">
      <input v-model="newComment" placeholder="Add a comment..." />
      <button @click="addComment">Add Comment</button>
      <button
        v-if="replyingToComment"
        @click="cancelReply"
        class="cancel-reply"
      >
        Cancel Reply
      </button>
    </div>
  </div>
</template>

<script setup>
import CommentItem from "./CommentItem.vue";
import { ref, onMounted, computed, watch } from "vue";
import { v4 as uuid } from "uuid";
import { debounce } from "lodash";

const isPlaying = ref(false);
const currentTime = ref(0);
const duration = ref(0);
const newComment = ref("");
// const comments = ref([]);
const errorMessage = ref("");
const WINDOW_SIZE_SECONDS = 120; // Size of the comment window in seconds

const commentWindowPosition = computed(() => {
  const windowStartTime = Math.max(
    0,
    currentTime.value - WINDOW_SIZE_SECONDS / 2
  );
  return (windowStartTime / duration.value) * 100;
});

const commentWindowWidth = computed(() => {
  return (WINDOW_SIZE_SECONDS / duration.value) * 100;
});

const visibleComments = ref([]);

const topLevelComments = computed(() => {
  return visibleComments.value.filter((comment) => !comment.parent);
});

const updateVisibleComments = () => {
  const windowStart = currentTime.value - WINDOW_SIZE_SECONDS / 2;
  const windowEnd = currentTime.value + WINDOW_SIZE_SECONDS / 2;
  visibleComments.value = comments.value
    .filter((comment) => {
      const commentTime = comment.time / 1000; // Convert milliseconds to seconds
      return commentTime >= windowStart && commentTime <= windowEnd;
    })
    .sort((a, b) => a.time - b.time);
};

const audio = new Audio();

const debouncedUpdateVisibleComments = debounce(updateVisibleComments, 100);
watch(currentTime, debouncedUpdateVisibleComments);

const seek = () => {
  audio.currentTime = currentTime.value;
  debouncedUpdateVisibleComments();
};

const togglePlay = () => {
  if (isPlaying.value) {
    audio.pause();
  } else {
    audio.play().catch((e) => {
      errorMessage.value = `Error playing audio: ${e.message}`;
      isPlaying.value = false;
    });
  }
  isPlaying.value = !isPlaying.value;
};

onMounted(() => {
  audio.addEventListener("timeupdate", () => {
    currentTime.value = audio.currentTime;
  });

  audio.addEventListener("seeking", debouncedUpdateVisibleComments);
  audio.addEventListener("seeked", debouncedUpdateVisibleComments);

  // Load the MPGA file
  loadAudio("RS_Episode_263.mpga");
});

const formatTime = (time) => {
  const minutes = Math.floor(time / 60);
  const seconds = Math.floor(time % 60);
  return `${minutes}:${seconds.toString().padStart(2, "0")}`;
};

const loadAudio = (src) => {
  audio.src = src;
  audio.load();
  audio.oncanplaythrough = () => {
    errorMessage.value = "";
    duration.value = audio.duration;
  };
  audio.onerror = (e) => {
    errorMessage.value = `Error loading audio: ${audio.error.message}`;
  };
};

onMounted(() => {
  audio.addEventListener("timeupdate", () => {
    currentTime.value = audio.currentTime;
  });

  // Load the MPGA file
  loadAudio("RS_Episode_263.mpga");
});

const comments = ref([
  {
    id: uuid(),
    author: "User1",
    text: "Great intro!",
    time: 0,
    createdAt: new Date() - 1000 * 60 * 25,
    parent: null,
  },

  {
    id: uuid(),
    author: "User2",
    text: "I am not sure about that",
    time: 1000 * 60 * 1,
    createdAt: new Date(),
    parent: null,
  },
  {
    id: uuid(),
    author: "DM",
    text: "I disagree",
    time: 1000 * 60 * 2,
    createdAt: new Date() - 1000 * 60 * 2,
    parent: null,
  },
  {
    id: uuid(),
    author: "otto",
    text: "What day is it?",
    time: 1000 * 60 * 5,
    createdAt: new Date() - 1000 * 60 * 5,
    parent: null,
  },
  {
    id: uuid(),
    author: "User3",
    text: "Why?",
    time: 1000 * 60 * 12,
    createdAt: new Date() - 1000 * 60 * 12,
    parent: null,
  },
]);

const replyingToComment = ref(null);

const replyToComment = (comment) => {
  replyingToComment.value = comment;
  newComment.value = `@${comment.author} `;
  // You might want to scroll to the comment input or focus it
};

const cancelReply = () => {
  replyingToComment.value = null;
  newComment.value = "";
};

const addComment = () => {
  if (newComment.value.trim()) {
    comments.value.push({
      id: uuid(),
      author: "Current User", // You might want to replace this with actual user info
      text: newComment.value,
      createdAt: new Date(),
      time: Math.floor(currentTime.value * 1000), // Convert to milliseconds
      parent: replyingToComment.value ? replyingToComment.value.id : null,
    });
    newComment.value = "";
    replyingToComment.value = null; // Reset the replying to comment
  }
};

const heatmapData = computed(() => {
  const heatmap = new Array(100).fill(0); // 100 segments for the heatmap
  comments.value.forEach((comment) => {
    const index = Math.floor((comment.time / (duration.value * 1000)) * 100);
    if (index >= 0 && index < 100) {
      heatmap[index]++;
    }
  });
  return heatmap;
});

const maxCommentCount = computed(() => Math.max(...heatmapData.value));
</script>

<style scoped>
.audio-player {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
}

.player-controls {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
}

.comment-input {
  display: flex;
  gap: 10px;
}

input[type="range"] {
  width: 100%;
}

.timeline {
  position: relative;
  height: 40px;
  background-color: #f0f0f0;
  margin-bottom: 20px;
}

.comment-window {
  position: absolute;
  top: 0;
  height: 100%;
  width: 10%; /* Adjust as needed */
  background-color: rgba(0, 123, 255, 0.3);
  pointer-events: none;
  transition: left 0.1s linear;
}

.comment-display {
  height: 150px;
  overflow-y: auto;
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 20px;
}

.comment {
  margin-bottom: 5px;
  padding: 5px;
  background-color: #f9f9f9;
  border-radius: 4px;
  position: relative;
}

.comment-content {
  margin-bottom: 5px;
}

.comment-actions {
  text-align: right;
}

.comment.replying-to {
  background-color: #e6f2ff;
  border: 1px solid #007bff;
}

.cancel-reply {
  margin-left: 10px;
  background-color: #f8f9fa;
  border: 1px solid #ced4da;
  color: #6c757d;
}

.cancel-reply:hover {
  background-color: #e2e6ea;
  color: #343a40;
}

.reply-link {
  font-size: 0.8em;
  color: #007bff;
  text-decoration: none;
}

.reply-link:hover {
  text-decoration: underline;
}

.heatmap {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: flex-end;
  pointer-events: none;
}

.heatmap-segment {
  flex-grow: 1;
  transition: height 0.3s ease;
}

.seek-bar {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  -webkit-appearance: none;
  background: transparent;
}

.seek-bar::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  background: #007bff;
  cursor: pointer;
  border-radius: 50%;
}

.seek-bar::-moz-range-thumb {
  width: 20px;
  height: 20px;
  background: #007bff;
  cursor: pointer;
  border-radius: 50%;
}
</style>
