<template>
  <div class="comment" :class="{ 'replying-to': isReplyingTo }">
    <div class="comment-header">
      <span class="comment-author">{{ comment.author }}</span>
      <span class="comment-time">{{
        formatCommentTime(comment.createdAt)
      }}</span>
    </div>
    <div class="comment-text">
      {{ comment.text }}
    </div>
    <div class="comment-actions">
      <a href="#" @click.prevent="$emit('reply', comment)" class="reply-link"
        >Reply</a
      >
    </div>
    <div v-if="replies.length > 0" class="replies">
      <CommentItem
        v-for="reply in replies"
        :key="reply.id"
        :comment="reply"
        :all-comments="allComments"
        :is-replying-to="isReplyingTo && replyingToComment.id === reply.id"
        @reply="$emit('reply', $event)"
      />
    </div>
  </div>
</template>

<script setup>
import { computed } from "vue";

// interface Comment {
//   id: number;
//   author: string;
//   time: number;
//   text: string;
//   parent?: number;
// }

const props = defineProps({
  comment: Object,
  allComments: Array,
  isReplyingTo: Boolean,
  replyingToComment: Object,
});

const emit = defineEmits(["reply"]);

const replies = computed(() => {
  return props.allComments.filter((c) => c.parent === props.comment.id);
});

const formatCommentTime = (time) => {
  const date = new Date(time);
  // get relative time
  const diff = Date.now() - date.getTime();
  if (diff < 1000 * 60) {
    return "Just now";
  } else if (diff < 1000 * 60 * 60) {
    return `${Math.floor(diff / (1000 * 60))} minutes ago`;
  } else if (diff < 1000 * 60 * 60 * 24) {
    return `${Math.floor(diff / (1000 * 60 * 60))} hours ago`;
  } else if (diff < 1000 * 60 * 60 * 24 * 7) {
    return `${Math.floor(diff / (1000 * 60 * 60 * 24))} days ago`;
  } else if (diff < 1000 * 60 * 60 * 24 * 30) {
    return `${Math.floor(diff / (1000 * 60 * 60 * 24 * 7))} weeks ago`;
  } else if (diff < 1000 * 60 * 60 * 24 * 365) {
    return `${Math.floor(diff / (1000 * 60 * 60 * 24 * 30))} months ago`;
  } else {
    return `${Math.floor(diff / (1000 * 60 * 60 * 24 * 365))} years ago`;
  }

  return date.toLocaleString(); // You can customize this format as needed
};
</script>

<style scoped>
.comment {
  margin-bottom: 10px;
  padding: 10px;
  background-color: #f9f9f9;
  border-radius: 4px;
}

.comment-header {
  margin-bottom: 5px;
}

.comment-author {
  font-size: 0.9em;
  font-weight: bold;
  color: #555;
}

.comment-time {
  font-size: 0.8em;
  color: #888;
  margin-left: 10px;
}

.comment-text {
  margin-bottom: 5px;
  font-size: 1em;
  color: #333;
}

.comment-actions {
  text-align: right;
}

.reply-link {
  font-size: 0.8em;
  color: #007bff;
  text-decoration: none;
}

.reply-link:hover {
  text-decoration: underline;
}

.replies {
  margin-left: 20px;
  border-left: 2px solid #e0e0e0;
  padding-left: 10px;
}

.replying-to {
  background-color: #e6f2ff;
  border: 1px solid #007bff;
}
</style>
