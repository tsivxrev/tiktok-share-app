<script setup>
import axios from 'axios';
import { reactive } from 'vue';

const state = reactive({
  isLoading: false,
  link: '',
  currentTimeout: 0,

  video: {
    status: 'нету)',
  },

  isError: false,
  error_title: '',
  error_message: '',
  error_details: '',
});

const error = (title, message, details) => {
  state.isError = true;
  state.error_title = title;
  state.error_message = message;
  state.error_details = details;
};

const urlRegex = /(https?):\/\/(.*)tiktok.com\/(.*)/;
const validateLink = () => {
  state.isError = false;

  if (urlRegex.test(state.link)) {
    clearTimeout(state.currentTimeout);
    state.currentTimeout = setTimeout(async () => {
      state.isLoading = true;
      state.isError = false;

      try {
        const { data } = await axios.get(`https://ttshare-api.tsivx.ru?url=${state.link}`);

        if (data.status !== 'success') {
          error('Видео недоступно', 'Если ты уверен в том что с видео всё хорошо, то проверь, правильно ли ты ввел ссылку.', data.reason);
        }

        state.isLoading = false;
        state.video = data;
      } catch (err) {
        state.isLoading = false;
        error('Произошли технические шоколадки :(', 'Не удалось отправить запрос на сервер', err);
      }
    }, 1000);
  }
};

window.state = state;
</script>

<template>
  <div class="main flex flex-col items-center h-screen w-screen px-4 py-6 gap-4 overflow-x-hidden">
    <div class="header w-full max-w-xl rounded-md">
      <div class="header-title text-lg font-bold text-white">
        TikTok Share (Beta)
      </div>
      <div class="header-subtitle text-neutral-500">
        Скачивай видео и музыку с TikTok без водяных знаков
      </div>
    </div>
    <div class="link w-full flex flex-col max-w-xl rounded-md">
      <label class="mb-2 text-sm font-medium text-neutral-500">Ссылка на видео</label>
      <input @input="validateLink" v-model="state.link" type="text" class="rounded-lg bg-neutral-800 placeholder:text-neutral-500 text-white w-full text-sm p-2.5" placeholder="https://vt.tiktok.com/ZSR2sAmMa/">
    </div>
    <div v-if="state.isLoading" class="spinner w-full max-w-xl flex flex-col items-center">
      <svg class="text-neutral-600 animate-spin" fill="none" height="24" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg">
        <path clip-rule="evenodd" d="M16.394 5.077A8.2 8.2 0 0 0 4.58 15.49a.9.9 0 0 1-1.628.767A10 10 0 1 1 12 22a.9.9 0 0 1 0-1.8 8.2 8.2 0 0 0 4.394-15.123z" fill="currentColor" fill-rule="evenodd"/>
      </svg>
    </div>
    <div v-if="state.isError" class="error w-full max-w-xl rounded-md text-white bg-neutral-800 px-4 py-2 flex flex-col">
      <div class="title text-lg font-bold">{{ state.error_title }}</div>
      <div class="message text-neutral-500">{{state.error_message}}</div>
      <div class="details font-mono text-sm text-neutral-500">{{state.error_details}}</div>
    </div>
    <div v-if="state.video.status === 'success'" class="video w-full max-w-xl flex-wrap rounded-md text-white bg-neutral-800 p-2 flex gap-3">
      <video controls class="video flex-grow w-full max-h-96 rounded-md" :src="state.video.nwm_video_url"></video>
      <div class="details flex flex-col gap-8 justify-between">
        <div class="top flex flex-col gap-2">
        <div class="title">{{ state.video.video_title }}</div>
        <div class="counters flex gap-1 flex-wrap">
          <div class="counter flex items-center gap-1 bg-rose-500 px-2 py-1 rounded-lg font-medium text-white">
            <svg class="h-4" viewBox="0 0 24 24" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
                <g id="Page-2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                    <g id="like_24">
                        <rect id="Bounds" x="0" y="0" width="24" height="24"></rect>
                        <path d="M16.7368421,3.10000002 C20.1406602,3.10000002 22.9,5.85933979 22.9,9.26315789 C22.9,12.6821066 21.5318842,14.3941802 15.7838129,18.8649023 L13.1664872,20.9006002 C12.4803772,21.4342412 11.5196228,21.4342412 10.8335128,20.9006002 L8.21618708,18.8649023 C2.46811582,14.3941802 1.10000002,12.6821066 1.10000002,9.26315789 C1.10000002,5.85933979 3.85933979,3.10000002 7.26315789,3.10000002 C9.07998925,3.10000002 10.6558981,4.16115038 12,6.18649354 C13.3441019,4.16115038 14.9200108,3.10000002 16.7368421,3.10000002 Z" id="Mask" fill="currentColor" fill-rule="nonzero"></path>
                    </g>
                </g>
            </svg>
            {{ state.video.video_digg_count }}
          </div>
          <div class="counter flex items-center gap-1 bg-neutral-700 px-2 py-1 rounded-lg font-medium text-white">
            <svg class="h-4" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M17.4361 5C18.3276 5 18.6509 5.09283 18.9768 5.26713C19.3028 5.44144 19.5586 5.69723 19.7329 6.02315C19.9072 6.34908 20 6.67237 20 7.56389V14.4361C20 15.3276 19.9072 15.6509 19.7329 15.9768C19.5586 16.3028 19.3028 16.5586 18.9768 16.7329C18.6509 16.9072 18.3276 17 17.4361 17H12.998L9.28041 20.7192C8.98756 21.0121 8.51269 21.0122 8.21975 20.7194C8.07905 20.5787 8 20.3879 8 20.189V17H6.56389C5.67237 17 5.34908 16.9072 5.02315 16.7329C4.69723 16.5586 4.44144 16.3028 4.26713 15.9768C4.09283 15.6509 4 15.3276 4 14.4361V7.56389C4 6.67237 4.09283 6.34908 4.26713 6.02315C4.44144 5.69723 4.69723 5.44144 5.02315 5.26713C5.34908 5.09283 5.67237 5 6.56389 5H17.4361Z" fill="currentColor"/>
            </svg>
            {{ state.video.video_comment_count }}
          </div>
          <div class="counter flex items-center gap-1 bg-neutral-700 px-2 py-1 rounded-lg font-medium text-white">
            <svg class="h-4" viewBox="0 0 24 24" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
                <g id="Page-2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                    <g id="share_24">
                        <polygon id="Shape" points="0 0 24 0 24 24 0 24"></polygon>
                        <path d="M13,8.75 L13,4.58122089 C13,4.30507851 13.2238576,4.08122089 13.5,4.08122089 C13.6186451,4.08122089 13.7334241,4.12341104 13.8238244,4.20025138 L22.5518006,11.619038 C22.7622042,11.7978813 22.787789,12.1134283 22.6089457,12.3238319 C22.5914704,12.3443911 22.5723595,12.363502 22.5518002,12.3809773 L13.823824,19.7997502 C13.6134202,19.9785933 13.2978733,19.9530083 13.1190302,19.7426044 C13.04219,19.6522042 13,19.5374253 13,19.4187804 L13,15.25 C7.6538669,15.25 3.86076798,16.394955 1.62070321,18.6848651 L1.62073341,18.6848947 C1.52416615,18.7836107 1.36585774,18.7853527 1.26714168,18.6887854 C1.20470735,18.6277101 1.17866622,18.5384835 1.19844738,18.4534133 C2.70265417,11.9844711 6.63650504,8.75 13,8.75 Z M13,8.75 L13,4.58122089 C13,4.30507851 13.2238576,4.08122089 13.5,4.08122089 C13.6186451,4.08122089 13.7334241,4.12341104 13.8238244,4.20025138 L22.5518006,11.619038 C22.7622042,11.7978813 22.787789,12.1134283 22.6089457,12.3238319 C22.5914704,12.3443911 22.5723595,12.363502 22.5518002,12.3809773 L13.823824,19.7997502 C13.6134202,19.9785933 13.2978733,19.9530083 13.1190302,19.7426044 C13.04219,19.6522042 13,19.5374253 13,19.4187804 L13,15.25 C7.6538669,15.25 3.86076798,16.394955 1.62070321,18.6848651 L1.62073341,18.6848947 C1.52416615,18.7836107 1.36585774,18.7853527 1.26714168,18.6887854 C1.20470735,18.6277101 1.17866622,18.5384835 1.19844738,18.4534133 C2.70265417,11.9844711 6.63650504,8.75 13,8.75 Z" id="Mask" fill="currentColor" fill-rule="nonzero"></path>
                    </g>
                </g>
            </svg>
            {{ state.video.video_share_count }}
          </div>
          <div class="counter flex items-center gap-1 bg-neutral-700 px-2 py-1 rounded-lg font-medium text-white">
            <svg class="h-4" viewBox="0 0 24 24" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
                <g id="Page-2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                    <g id="view_24">
                        <rect id="icon-bounds" x="0" y="0" width="24" height="24"></rect>
                        <path d="M12,19 C6,19 2,13.4 2,12 C2,10.6 6,5 12,5 C18,5 22,10.6 22,12 C22,13.4 18,19 12,19 Z M12,17 C14.7614237,17 17,14.7614237 17,12 C17,9.23857625 14.7614237,7 12,7 C9.23857625,7 7,9.23857625 7,12 C7,14.7614237 9.23857625,17 12,17 Z M12.0009513,14.5 C10.6202394,14.5 9.50095131,13.3807119 9.50095131,12 C9.50095131,10.6192881 10.6202394,9.5 12.0009513,9.5 C13.3816632,9.5 14.5009513,10.6192881 14.5009513,12 C14.5009513,13.3807119 13.3816632,14.5 12.0009513,14.5 Z" id="Mask" fill="currentColor"></path>
                    </g>
                </g>
            </svg>
            {{ state.video.video_play_count }}
          </div>
          <div class="counter flex items-center gap-1 bg-neutral-700 px-2 py-1 rounded-lg font-medium text-white">
            <svg class="h-4" viewBox="0 0 24 24" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
                <g id="Page-2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                    <g id="download_24">
                        <polygon id="Shape" points="0 0 24 0 24 24 0 24"></polygon>
                        <path d="M9,3 L15,3 L15,9 L18.0343146,9 C18.2552285,9 18.4343146,9.1790861 18.4343146,9.4 C18.4343146,9.5060866 18.3921718,9.60782816 18.3171573,9.68284271 L12.2828427,15.7171573 C12.126633,15.873367 11.873367,15.873367 11.7171573,15.7171573 L5.68284271,9.68284271 C5.526633,9.526633 5.526633,9.273367 5.68284271,9.11715729 C5.75785726,9.04214274 5.85959883,9 5.96568542,9 L9,9 L9,3 Z M6,18 L18,18 C18.5522847,18 19,18.4477153 19,19 L19,19 C19,19.5522847 18.5522847,20 18,20 L6,20 C5.44771525,20 5,19.5522847 5,19 L5,19 C5,18.4477153 5.44771525,18 6,18 Z" id="Mask" fill="currentColor"></path>
                    </g>
                </g>
            </svg>
            {{ state.video.video_download_count }}
          </div>
        </div>
        <div class="music flex items-center gap-1">
          <svg class="h-5" viewBox="0 0 24 24" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
              <g id="Page-2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                  <g id="music_24">
                      <polygon id="Bounds" points="0 0 24 0 24 24 0 24"></polygon>
                      <path d="M19,10.0330156 L19,14.4855012 C19,17.956963 17.7640906,18.7050403 15.7191978,18.9799957 C14.6568486,19.1228388 13,18.5657522 13,16.5476411 C13,15.5618364 13.8210463,14.5825028 14.8458571,14.3810547 C15.7962799,14.1942291 14.5376081,14.4385026 16.6877482,14.029047 C17.0341131,13.963088 17.0048905,13.7164073 17.0048905,13.4491281 C17.0048905,13.2322124 17.003783,11.2499769 17.0030913,10.0330156 L17,10.0330156 L17.0048905,7.45692129 C17.0048905,7.22036495 16.9318183,7.10889704 16.6599628,7.17689001 L9.36450452,8.50635031 C9.05986192,8.58254359 9,8.70905871 9,9 L9,11.5017817 L9,16.4855012 C9,19.956963 7.76409063,20.7050403 5.71919783,20.9799957 C4.65684865,21.1228388 3,20.5657522 3,18.5476411 C3,17.5618364 3.82104626,16.5825028 4.84585712,16.3810547 C5.79627991,16.1942291 4.53760807,16.4385026 6.68774818,16.029047 C7.03411311,15.963088 7.00489048,15.7164073 7.00489048,15.4491281 C7.00489048,15.1337045 7.00254873,11.0855243 7.00254873,11.0855243 L7.00420079,11.0854536 L7,8.54964861 C7,8.54964861 6.99591107,6.97596748 7,6.18913492 C7.00489048,5.24805968 7.51028442,4.90220642 9,4.59580161 L17,3.11819807 C17,3.11819807 19,2.49293028 19,4.11862654 L19,10.0330156 Z" id="Mask" fill="currentColor"></path>
                  </g>
              </g>
          </svg>
          <span>{{ state.video.video_music_title }}</span>
        </div>
        <div class="author flex items-center gap-1">
          <svg class="h-5" viewBox="0 0 24 24" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
              <g id="Page-2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                  <g id="user_24">
                      <polygon id="Bounds" points="0 0 24 0 24 24 0 24"></polygon>
                      <path d="M16,8 C16,5.79 14.21,4 12,4 C9.79,4 8,5.79 8,8 C8,10.21 9.79,12 12,12 C14.21,12 16,10.21 16,8 Z M4,18 L4,18.995 C4,19.5500462 4.44995383,20 5.005,20 L18.995,20 C19.5500462,20 20,19.5500462 20,18.995 L20,18 C20,14.5 14.67,13.5 12,13.5 C9.33,13.5 4,14.5 4,18 Z" id="Mask" fill="currentColor"></path>
                  </g>
              </g>
          </svg>
          <span>@{{ state.video.video_author_id }}</span>
        </div>
        </div>
        <div class="bottom">
          <div class="download-buttons flex flex-col gap-1">
            <a download target="_blank" class="ease-linear duration-100 font-medium flex px-2 py-1 rounded-lg gap-1 bg-green-500 hover:bg-green-600 items-center" :href="state.video.nwm_video_url">
              <svg class="h-5" viewBox="0 0 24 24" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
                  <g id="Page-2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                      <g id="download_24">
                          <polygon id="Shape" points="0 0 24 0 24 24 0 24"></polygon>
                          <path d="M9,3 L15,3 L15,9 L18.0343146,9 C18.2552285,9 18.4343146,9.1790861 18.4343146,9.4 C18.4343146,9.5060866 18.3921718,9.60782816 18.3171573,9.68284271 L12.2828427,15.7171573 C12.126633,15.873367 11.873367,15.873367 11.7171573,15.7171573 L5.68284271,9.68284271 C5.526633,9.526633 5.526633,9.273367 5.68284271,9.11715729 C5.75785726,9.04214274 5.85959883,9 5.96568542,9 L9,9 L9,3 Z M6,18 L18,18 C18.5522847,18 19,18.4477153 19,19 L19,19 C19,19.5522847 18.5522847,20 18,20 L6,20 C5.44771525,20 5,19.5522847 5,19 L5,19 C5,18.4477153 5.44771525,18 6,18 Z" id="Mask" fill="currentColor"></path>
                      </g>
                  </g>
              </svg>
            Видео без водяного знака
            </a>
            <a download target="_blank" class="ease-linear duration-100 font-medium flex px-2 py-1 rounded-lg gap-1 hover:bg-neutral-700 items-center" :href="state.video.wm_video_url">
              <svg class="h-5" viewBox="0 0 24 24" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
                  <g id="Page-2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                      <g id="download_24">
                          <polygon id="Shape" points="0 0 24 0 24 24 0 24"></polygon>
                          <path d="M9,3 L15,3 L15,9 L18.0343146,9 C18.2552285,9 18.4343146,9.1790861 18.4343146,9.4 C18.4343146,9.5060866 18.3921718,9.60782816 18.3171573,9.68284271 L12.2828427,15.7171573 C12.126633,15.873367 11.873367,15.873367 11.7171573,15.7171573 L5.68284271,9.68284271 C5.526633,9.526633 5.526633,9.273367 5.68284271,9.11715729 C5.75785726,9.04214274 5.85959883,9 5.96568542,9 L9,9 L9,3 Z M6,18 L18,18 C18.5522847,18 19,18.4477153 19,19 L19,19 C19,19.5522847 18.5522847,20 18,20 L6,20 C5.44771525,20 5,19.5522847 5,19 L5,19 C5,18.4477153 5.44771525,18 6,18 Z" id="Mask" fill="currentColor"></path>
                      </g>
                  </g>
              </svg>
            Видео с водяным знаком (фу)
            </a>
            <a download target="_blank" class="ease-linear duration-100 font-medium flex px-2 py-1 rounded-lg gap-1 hover:bg-neutral-700 items-center" :href="state.video.video_music_url">
              <svg class="h-5" viewBox="0 0 24 24" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
                  <g id="Page-2" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                      <g id="download_24">
                          <polygon id="Shape" points="0 0 24 0 24 24 0 24"></polygon>
                          <path d="M9,3 L15,3 L15,9 L18.0343146,9 C18.2552285,9 18.4343146,9.1790861 18.4343146,9.4 C18.4343146,9.5060866 18.3921718,9.60782816 18.3171573,9.68284271 L12.2828427,15.7171573 C12.126633,15.873367 11.873367,15.873367 11.7171573,15.7171573 L5.68284271,9.68284271 C5.526633,9.526633 5.526633,9.273367 5.68284271,9.11715729 C5.75785726,9.04214274 5.85959883,9 5.96568542,9 L9,9 L9,3 Z M6,18 L18,18 C18.5522847,18 19,18.4477153 19,19 L19,19 C19,19.5522847 18.5522847,20 18,20 L6,20 C5.44771525,20 5,19.5522847 5,19 L5,19 C5,18.4477153 5.44771525,18 6,18 Z" id="Mask" fill="currentColor"></path>
                      </g>
                  </g>
              </svg>
            Музыка
            </a>
          </div>
        </div>
      </div>
    </div>
    <div class="footer flex items-center gap-4 text-neutral-500">
      <div>developed with love by <a target="_blank" class="font-medium text-white" href="https://vk.com/tsivxrev">@tsivxrev</a></div>
    </div>
  </div>
</template>

<style>
</style>
