<template>
    <div>
        <header class=" relative mb-8 relative">
            <div class=" relative ">

                <img class="w-full fill-current rounded-lg object-cover mb-2 h-48" src="/images/coverphoto.jpg"
                     alt="cover photo">
                <img :src="user.avatar" alt="user's avatar"
                     class="
                         rounded-full w-32 h-32
                         bottom-0
                         border-2
                         border-white
                         object-cover
                         absolute
                         -mb-16
                         -translate-x-1/2"
                     style="left:50%">

            </div>


            <div class="flex  justify-between items-center">
                <div style="max-width: 270px">
                    <h2 class="font-bold text-2xl">{{user.name}}</h2>
                    <p class="text-sm">Joined {{user.created_at | diffForHumans}}</p>
                    <div class=" my-3 text-gray-600  flex space-x-2 ">
                        <p class="text-sm">Followers {{followers}} |</p>
                        <p class="text-sm">Following {{followings}}</p></div>
                </div>
                <div class="space-x-2 flex ">

                    <slot></slot>

                </div>
            </div>
            <div v-if="editable" class="flex flex-col items-center transition -translate-y-12">
                <div v-if="this.bio.length" class="items-center w-1/2 ">
                    <p class="text-sm p-2 my-4  items-center text-center text-gray-600">{{this.bio}}</p>
                    <div class="flex space-x-2">
                        <button v-if="!bioFieldShowed" @click="showBioField()"
                                class="text-sm text-gray-600 border-b-2  hover:border-blue-400 focus:outline-none outline-none justify-center flex items-center w-full">
                            Edit bio
                        </button>
                        <button v-if="!bioFieldShowed" @click="confirmationModel = true"
                                class="text-sm text-gray-600 border-b-2  hover:border-red-400 focus:outline-none outline-none justify-center flex items-center w-full">
                            Remove bio
                        </button>
                    </div>
                </div>
                <div v-else class="flex w-full items-center transition translate-y-3 p-2 m-4 w-full justify-center">
                    <button v-if="!bioFieldShowed" @click="showBioField()"
                            class="text-gray-600 border-b-2  hover:border-blue-400 focus:outline-none outline-none justify-center flex items-center w-1/2">
                        Add bio
                    </button>
                </div>
                <div v-if="bioFieldShowed"
                     class="flex w-full items-center transition translate-y-3 p-2 mb-4 w-full justify-center">
                    <form @submit.prevent="handleBioSubmite()"
                          class="flex w-full items-center transition translate-y-3 p-2 m-4 w-full justify-center">
                                    <textarea v-model="bio" type="textarea"
                                              placeholder="write your bio here"
                                              name="bio"
                                              id="bio"
                                              v-autofocus
                                              class="w-full border-gray-300 border-b-2 w-full focus:outline-none outline-none focus:border-blue-400"
                                              maxlength="200">
                                   </textarea>

                        <button
                            class="
                                  absolute
                                  right-0 text-sm  mx-8 focus:outline-none transition -translate-y-4"
                            :class="totalCharacter > 0 ? 'text-white font-medium bg-blue-400  rounded-full px-2 py-1 shadow hover:bg-gray-100 hover:text-blue-500'  : 'text-gray-400'"
                            :disabled="totalCharacter<=0" type="submit"
                        >Save Bio
                        </button>
                        <vue-countable :text="bio"   :elementId="'bio'" @change="change">

                        </vue-countable>
                        <button
                            class="
                                  absolute
                                  right-0 text-sm  mx-24 focus:outline-none transition -translate-y-4
                                  text-gray-700 px-3 py-1 btn-link px-6 "

                            @click="cancleBio()"
                        >Cancle
                        </button>
                    </form>
                    <span
                        class="flex-shrink-0 mx-2 text-gray-500 bottom-0 transition translate-y-4 text-sm absolute right-0">{{ totalCharacter }} / 200</span>
                </div>
            </div>
            <div v-else-if="user.bio" class="flex justify-center transition -translate-y-12">
                <p class="text-sm p-2 my-4  items-center text-center text-justify text-gray-600">
                    {{this.bio}}
                </p>
            </div>
        </header>

        <div v-if="confirmationModel === true">
            <modal @close="confirmationModel = false">
                You are about to delete your bio?
                <template slot="footer">
                    <div class="flex space-x-4 mx-4">
                        <button
                            @click="handleRemoveBio()"
                            class="  focus:outline-none outline-none text-gray-600 border-2
                            border-gray-400 rounded-full py-2 px-4
                            text-lg hover:bg-gray-300 hover:text-gray-800
                              font-bold">
                            Sure, delete it
                        </button>
                        <button @click="confirmationModel = false"
                                class="focus:outline-none outline-none
                                text-gray-600 btn-link text-lg
                                font-bold  hover:text-blue-600">
                            Cancel
                        </button>
                    </div>
                </template>
            </modal>
        </div>
    </div>
</template>

<style>
    @import '~animate.css';
</style>

<script>

    export default {
        components: {},
        props: {
            user: {required: true},
            followings : {required:true},
            followers: {require:true},
            isCurrentUser: {
                default: () => {
                }, required: true
            },
            editable: {
                required: true
            }
        },
        data: () => ({
            userInfo: '',
            title: 'this is title',
            bio: '',
            totalCharacter: 0,
            bioFieldShowed: false,
            confirmationModel: false
        }),
        created() {
            this.getUserBio()

        },
        filters: {
            diffForHumans: (date) => {
                if (!date) {
                    return null;
                }
                return dayjs(date).fromNow();
            }
        },

        methods: {
            getUserBio() {
                axios.get(`/bio/${this.user.username}`).then((response) => {
                    this.clearifyBio(response.data)
                }).catch(error => console.log(error))
            },

            charCount() {
                this.totalCharacter = this.bio.length;
            },

            showBioField() {
                this.bioFieldShowed = true;
            },

            handleBioSubmite() {
                axios.patch(`/bio/${this.user.username}/edit`, {bio: this.bio}).then((response) => {
                    this.getUserBio(),
                        this.bioFieldShowed = false
                }).catch(error => console.log(error))
            },

            clearifyBio(userBio) {
                if (userBio.length) {
                   return  this.bio = userBio;
                }
                return this.bio = ""
            },

            handleRemoveBio() {
                axios.delete(`/bio/${this.user.username}/delete`).then(() => {
                    this.getUserBio();
                    this.confirmationModel = false;
                }).catch(error => console.log(error))
            },

            cancleBio() {
                this.bioFieldShowed = false;
                this.getUserBio();
                this.totalCharacter = 0;
             },

             change(event) {
                this.totalCharacter = event.all
             }
        }
    }
</script>
