<script lang="ts">
  import { title } from "../libs/ui";
  import { GAME } from "../libs/sdk";
  import type { GenericRanking } from "../libs/generalTypes";
  import StatusOverlays from "../components/StatusOverlays.svelte";
  import type { GameName } from "../libs/scoring";
  import { GAME_TITLE } from "../libs/i18n";
  import { t } from "../libs/i18n";

  export let game: GameName = 'mai2';

  title(`Ranking`);

  let d: { users: GenericRanking[] };
  let error: string | null;
  Promise.all([GAME.ranking(game)])
    .then(([users]) => {
      console.log(users)
      d = { users };
    })
    .catch((e) => error = e.message);
</script>

<main class="content leaderboard">
  <div class="outer-title-options">
    <h2>{t("Leaderboard.Title")}</h2>
    <nav>
      {#each Object.entries(GAME_TITLE) as [k, v]}
        <a href="/ranking/{k}" class:active={k === game}>{v}</a>
      {/each}
    </nav>
  </div>

  {#if d}
    <div class="leaderboard-container">
      <div class="lb-user">
        <span class="rank">{t("Leaderboard.Rank")}</span>
        <span class="name"></span>
        <span class="rating">{t("Leaderboard.Rating")}</span>
        <span class="accuracy">{t("Leaderboard.Accuracy")}</span>
        <span class="fc">{t("Leaderboard.FC")}</span>
        <span class="ap">{t("Leaderboard.AP")}</span>
      </div>
      {#each d.users as user, i (user.rank)}
        <div class="lb-user" class:alternate={i % 2 === 1}>
          <span class="rank">#{user.rank}</span>
          <span class="name">
            {#if user.username !== ""}
              <a href="/u/{user.username}/{game}" class:registered={!(/user\d+/.test(user.username))}>{user.name}</a>
            {:else}
              <span>{user.name}</span>
            {/if}
          </span>
          <span class="rating">{user.rating.toLocaleString()}</span>
          <span class="accuracy">{(+user.accuracy).toFixed(2)}%</span>
          <span class="fc">{user.fullCombo}</span>
          <span class="ap">{user.allPerfect}</span>
        </div>
      {/each}
    </div>
  {/if}

  <StatusOverlays error={error} loading={!d} />
</main>

<style lang="sass">
  @import "../vars"

  .leaderboard-container
    display: flex
    flex-direction: column

  .lb-user
    display: flex
    align-items: center
    justify-content: space-between
    width: 100%
    gap: 12px
    border-radius: $border-radius
    padding: 6px 12px
    box-sizing: border-box

    > *:not(.name)
      text-align: center

    .name
      min-width: 100px
      flex: 1

      > a
        color: unset

      .registered
        background: $grad-special
        color: transparent
        -webkit-background-clip: text
        background-clip: text

    .accuracy, .rating
      width: 15%
      min-width: 45px

    .rating
      font-weight: bold
      color: white

    .fc, .ap
      width: 5%
      min-width: 20px

    @media (max-width: $w-mobile)
      font-size: 0.9rem

      .accuracy
        display: none

    &.alternate
      background-color: $ov-light


</style>
