<template>
  <div class="controls">
    <div class="spanHo">
      <span
        v-for="currentUser in value"
        :key="currentUser.user"
        class="spann"
      >{{ currentUser.user }}
        <i
          class="icon-cancel"
          @click="removeUser(currentUser)"
        />
      </span>
    </div>
    <template v-if="showAddUsersComp">
      <div>{{ text }}</div>
      <input
        v-model="search"
        type="search"
        class="input"
        placeholder="Search"
        title="Filter by username"
      >
      <ul>
        <li
          v-for="user in filteredUsers"
          :key="user.user"
          @click="addUser(user)"
        >
          {{ user.user }}
        </li>
      </ul>
    </template>
  </div>
</template>
<script lang="ts">
import {Component, Prop, Vue} from 'vue-property-decorator';
import {CurrentUserInfoModel, UserModel} from '@/types/model';

@Component
export default class AddUserToRoom extends Vue {

  public search: string = '';

  @Prop() public value!: UserModel[];
  @Prop() public text!: string;
  @Prop() public excludeUsersIds!: number[];
  @Prop({default: true}) public showInviteUsers!: boolean;

  public removeUser(currentUser: UserModel) {
    this.value.splice(this.value.indexOf(currentUser), 1);
  }

  get users(): UserModel[] {
    const uids: number[] = this.value.map(a => a.id);
    uids.push(this.store.userInfo!.userId);
    uids.push(...this.excludeUsersIds);
    const users: UserModel[] = [];
    this.store.usersArray.forEach(u => {
      if (uids.indexOf(u.id) < 0) {
        users.push(u);
      }
    });
    this.logger.debug('Reeval users in CreatePrivateRoom')();

    return users;
  }

  get showAddUsersComp() {
    return this.showInviteUsers && this.users.length > 0;
  }

  public addUser(user: UserModel) {
    this.search = '';
    this.value.push(user);
  }

  get filteredUsers(): UserModel[] {
    this.logger.debug('Reeval filter CreatePrivateRoom')();
    const s = this.search.toLowerCase();

    return this.users.filter(u => u.user.toLowerCase().indexOf(s) >= 0);
  }
}
</script>

<style lang="sass" scoped>

  @import "~@/assets/sass/partials/abstract_classes"
  .spann
    @extend %hovered-user-room

  .controls
    display: flex
    flex-direction: column
    > *
      margin-bottom: 5px

  .icon-cancel
    cursor: pointer

  .color-reg .icon-cancel
      @include hover-click($red-cancel-reg)
  .color-lor .icon-cancel
    color: #a94442

  .spanHo
    max-width: 300px
    max-height: calc(50vh - 100px)
    overflow: auto

  ul
    min-height: 50px
    max-height: calc(50vh - 100px)
    margin-top: 5px
    overflow-y: scroll
    padding-left: 0

  li
    padding: 0 0 0 5px
    border-radius: 2px
    text-overflow: ellipsis
    overflow: hidden
    white-space: nowrap
    @extend %hovered-user-room
</style>
