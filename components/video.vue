<template>
  <div>
    <div class="prepare-container" :class="{ '-is-join': isJoin }">
      <div class="container">
        <div
          class="left"
          id="body"
          :class="[
            { '-no-member': !hasMember },
            { '-is-comment': isComment || isUsers },
          ]"
        >
          <video class="video" id="webcam-video" style="display: none"></video>
          <canvas id="landmarks" style="display: none"></canvas>
          <p v-if="isJoin" id="maximize-screen-user-name">あなた</p>
        </div>
        <div class="right" v-if="!isJoin">
          <p class="title">{{ room.name }}</p>
          　<button class="common-button" @click="$emit('click')">
            今すぐ参加する
          </button>
          <div class="avatar-select">
            　<button class="common-button -outline" @click="onSelect">
              アバターを選ぶ
            </button>
          </div>
        </div>
        <div class="remote-streams" id="js-remote-streams">
          <div class="right" v-if="hasMember"></div>
        </div>
        <Comment
          @close="onClose"
          @submit="onSubmit"
          v-if="isComment"
          :comments="comments"
        ></Comment>
        <UsersList @close="onClose" v-if="isUsers" :users="users"></UsersList>
      </div>
      <VideoFooter
        v-if="isJoin"
        :switch-scree-sharing="switchScreeSharing"
        @leave="$emit('leave')"
        @mute="$emit('mute', $event)"
        @video="$emit('video', $event)"
        @screen-sharing="$emit('screen-sharing', $event)"
        @comment="onClickComment"
        @users="onClickUsers"
      >
      </VideoFooter>
    </div>
    <AvatarSelect @click="onClickSelect" v-if="isSelecting"></AvatarSelect>
  </div>
</template>

<script>
import { defineComponent, ref, watch } from "@nuxtjs/composition-api";
import VideoFooter from "@/components/videoFooter";
import AvatarSelect from "@/components/AvatarSelect";
import Comment from "@/components/Comment";
import UsersList from "@/components/UsersList";

import { createText } from "@/compositions/useTextAnimation";

export default defineComponent({
  props: {
    isJoin: {
      type: Boolean,
      required: true,
    },
    hasMember: {
      type: Boolean,
      required: true,
    },
    switchScreeSharing: {
      type: Boolean,
    },
    room: {
      type: Object,
      required: true,
    },
    comments: {
      type: Array,
      required: true,
    },
    users: {
      type: Array,
      required: true,
    },
  },
  emits: [
    "click",
    "select-avatar",
    "leave",
    "mute",
    "screen-sharing",
    "video",
    "comment",
  ],
  components: [VideoFooter, AvatarSelect],
  setup(props, { emit }) {
    isSelecting;
    const isSelecting = ref(false);
    const isComment = ref(false);
    const isUsers = ref(false);

    const onSelect = () => {
      isSelecting.value = true;
    };
    const onClickSelect = (event) => {
      isSelecting.value = false;
      emit("select-avatar", event);
    };
    const onClickComment = () => {
      isComment.value = !isComment.value;
      isUsers.value = false;
    };
    const onClose = () => {
      isComment.value = false;
      isUsers.value = false;
    };
    const onSubmit = (inputValue) => {
      emit("comment", inputValue);
    };
    const onClickUsers = () => {
      isUsers.value = !isUsers.value;
      isComment.value = false;
    };
    watch(
      () => props.comments,
      async () => {
        const lastIndex = props.comments.length - 1;
        await createText(props.comments[lastIndex]);
        return;
      }
    );

    return {
      isSelecting,
      onSelect,
      onClickSelect,
      onClickComment,
      isComment,
      onClose,
      onClickUsers,
      onSubmit,
      isUsers,
    };
  },
});
</script>
