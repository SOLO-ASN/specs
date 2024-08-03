

|                     | Folder   | File                                                         | Function                                                     | Remarks           |
| ------------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------- |
| assets              | api      | assets/api/addressAssets.js                                  | Interacts with third-party APIs to retrieve assets on multiple blockchains |  |
|                     | images   | -                                                            | -                                                            |                  |
|                     | scss     | -                                                            | -                                                            |                  |
|                     | text     | assets/text/url.js                                           | Configuration parameters: fidoUrl: fido backend server URL, SoloMission API backend asnUrl: ASN official website |                  |
| components/Airdrops | cards    | 1. FidoListCard.vue 2. ProfileCard.vue 3. ProfileSettingCard.vue 4. SocialSettingCard.vue 5. WalletNetworkBox.vue 6. WalletNetworkMoreBox.vue 7. WalletSettingCard.vue 8. badge.vue 9. collection-card.vue 10. nft-card.vue 11. txs-card.vue | 1. Fido list component 2. User profile component 3. User profile settings component 4. User profile settings social account component 5. Wallet on-chain address addition component 6. Wallet on-chain address component 7. Wallet settings component 8. -- 9. -- 10. Nft list component 11. Transaction history component |                  |
|                     | profile  | 1. AccountSetting.vue 2. Gallery.vue 3. Gallery_space.vue 4. Sidebar.vue | 1. Account settings component 2. Gallery component 3. Gallery_space component 4. Sidebar component |                  |
|                     | --       | AuthFrame.vue                                                | Login and register functionality, integrated with FIDO login and registration, device query |                  |
|                     |          | CampaignCard.vue                                             | Campaign component                                             |                  |
|                     |          | ClaimButton.vue                                              | Claim button                                                 |                  |
|                     |          | ConfirmModal.vue                                              | Confirm component                                             |                  |
|                     |          | Filter.vue                                                   | Filter selection component                                    |                  |
|                     |          | Leaderboard.vue                                              | --                                                           |                  |
|                     |          | Login.vue                                                    | Login component                                               |                  |
|                     |          | LoginPrompt.vue                                              | Login prompt                                                 |                  |
|                     |          | Register.vue                                                 | Registration component                                        |                  |
|                     |          | SocialAuth.vue                                               | Adds login and register functionality, integrated with FIDO login and registration, device query |                  |
|                     |          | Sorter.vue                                                   | Modifies the redirection logic, improves the follow function |                  |
|                     |          | SpaceCard.vue                                                | Space component                                              |                  |
|                     |          | SpaceCreate.vue                                              | Space creation component                                      |                  |
|                     |          | SpaceDetail.vue                                              | Space display component                                       |                  |
|                     |          | Statistic.vue                                                | --                                                           |                  |
|                     |          | TabCategory.vue                                              | --                                                           |                  |
|                     |          | Tag.vue                                                      | --                                                           |                  |
|                     |          | readmore.vue                                                 | --                                                           |                  |
| components/Campaign | Button   | 1. Single.vue 2. Standard.vue 3. Tweet.vue 4. Verify.vue      | 1. Adds functionality to verify if the campaign is completed 2. Implements tweet sending and returning to the original campaign page 3. Initial tweet and verification functions 4. Verification |                  |
|                     | Card     | 1. NftCard.vue 2. PhotoCard.vue                             | 1. Adds campaign page 2. Adds reward image display on the Campaign page |                  |
|                     | Contact  | --                                                           | --                                                           |                  |
|                     | Faq      | --                                                           | --                                                           |                  |
|                     | Product  | --                                                           | --                                                           |                  |
|                     | Reward   | 1. Gallery.vue 2. MediaBanner.vue                           | 1. Sets the default display picture for tokens 2. Campaign supports dynamic reading of reward types |                  |
|                     | Twitter  | 1. Feature.vue                                               | Implements tweet sending and returning to the original campaign page |                  |
| pages               | id       | 1. accountSettings.vue 2. avatar.vue 3. profile.vue 4. recentTransactions.vue | 1. User settings page 2. Header avatar 3. Profile display page 4. -- |                  |
|                     | spaces   | space.vue                                                    | Space page                                                  |                  |
|                     | campaign | campaign-create.vue                                          | Create campaign                                              | 2 months ago     |
|                     |          | campaign-twitter.vue                                         | Campaign-twitter page                                        | 2 months ago     |
|                     |          | campaign.vue                                                 | Campaign task page                                          |                  |
