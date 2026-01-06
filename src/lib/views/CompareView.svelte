<script lang="ts" module>
    export const imageRoot = 'https://api.dofusdb.fr/img/'
    export const profilePictures = [
        'breeds/symbol_1.png',
        'breeds/symbol_2.png',
        'breeds/symbol_3.png',
        'breeds/symbol_4.png',
        'breeds/symbol_5.png',
        'breeds/symbol_6.png',
        'breeds/symbol_7.png',
        'breeds/symbol_8.png',
        'breeds/symbol_9.png',
        'breeds/symbol_10.png',
        'breeds/symbol_11.png',
        'breeds/symbol_12.png',
        'breeds/symbol_13.png',
        'breeds/symbol_14.png',
        'breeds/symbol_15.png',
        'breeds/symbol_16.png',
        'breeds/symbol_17.png',
        'breeds/symbol_18.png',
        'breeds/symbol_20.png',
    ]
</script>
<script lang="ts">
    import Row from '../components/Row.svelte'
    import Column from '../components/Column.svelte'
    import Text from '../components/Text.svelte'
    import Window from '../components/Window.svelte'
    import {
        profiles,
        type Profile,
        type ProfileData,
        getProfileProgress,
    } from '../data/state.svelte'
    import Accordion from '../components/Accordion.svelte'
    import ProfileChip from '../components/ProfileChip.svelte'
    let ownProfiles: ProfileData = $state({
        current: '',
        profiles: [],
        version: 1,
    })
    profiles.subscribe((p) => (ownProfiles = p))

    let selectedProfiles = $state<Profile[]>([])

    function toggleProfile(profile: Profile) {
        const exists = selectedProfiles.some(p => p.id === profile.id)
        if (exists) {
            selectedProfiles = selectedProfiles.filter(p => p.id !== profile.id)
        } else {
            selectedProfiles = [...selectedProfiles, profile]
        }
    }


    let displayedQuests = $state<null | Record<string, string[]>>(null)
    let displayedAchievements = $state<null | Record<string, string[]>>(null)

    type ProfileProgress = {
        quests: string[]
        achievements: string[]
    }

    const getSplittedProfilesProgress = (selectedProfiles: Profile[]): Record<string, ProfileProgress> => {
        const result = {} as Record<string, ProfileProgress>
        for (const profileId of selectedProfiles.map(p => p.id)) {
            const data = getProfileProgress(profileId)
            result[profileId] = {
                quests: data.filter((id: string) => id.startsWith('q')).map((id: string) => id.slice(1)),
                achievements: data.filter((id: string) => id.startsWith('a')).map((id: string) => id.slice(1))
            }
        }
        return result
    }

    const compare = (selectedProfiles: Profile[]) => {
        const progress = getSplittedProfilesProgress(selectedProfiles)
        const questsResult = {} as Record<string, string[]>
        const achievementsResult = {} as Record<string, string[]>
        const defaultProfileIds = selectedProfiles.map((profile) => profile.id)

        for (const [profileId, {quests, achievements}] of Object.entries(progress)) {
            for (const quest of quests) {
                if (!questsResult[quest]) questsResult[quest] = [...defaultProfileIds]
                questsResult[quest].splice(questsResult[quest].findIndex((id) => profileId == id), 1)
            }
            for (const achievement of achievements) {
                if (!achievementsResult[achievement]) achievementsResult[achievement] = [...defaultProfileIds]
                achievementsResult[achievement].splice(achievementsResult[achievement].findIndex((id) => profileId == id), 1)
            }
        }

        displayedQuests = Object.fromEntries(
            Object.entries(questsResult).filter(
                ([_, ids]) => ids.length !== 0
            )
        )

        displayedAchievements = Object.fromEntries(
            Object.entries(achievementsResult).filter(
                ([_, ids]) => ids.length !== 0
            )
        )
    }

</script>

{#snippet profile(profile: Profile, onclick?: () => void, selected = false)}
    <button
        class="profile profile-colored"
        class:noclick={onclick === undefined}
        class:selected={selected}
        {onclick}
        style:--color={profile.color}
    >
        <div class="image-box">
            <img
                src={imageRoot + profilePictures[profile.image]}
                alt=""
                loading="lazy"
            />
        </div>
        <div class="info">
            <div class="name">
                {profile.name.trim()}
            </div>
            <div class="title">
                <Text key={'t' + profile.title} name={profile.gender} />
            </div>
        </div>
    <!-- Add checked icon on selected-->
        {#if selected}
            <svg
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 24 24"
                fill="white"
                width="16px"
                height="16px"
                style="position: absolute; top: 4px; right: 4px;"
            >
                <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z" />
            </svg>
        {/if}
    </button>
{/snippet}

<Window id="compare" name={{ key: 'compare' }} maxWidth={800}>
    <Text element="h2" key="comparator"  />
    <Text key="compare-info" />
    <Row>
        <Column grow={1}>
            <Text element="h4" key="select-profiles"  />
            {#each ownProfiles.profiles as p}
                <Row classes="profile-row">
                    {@render profile(p, () => toggleProfile(p), selectedProfiles.some(sp => sp.id === p.id)) }
                </Row>
            {/each}
            <button disabled={selectedProfiles.length < 2} onclick={() => compare(selectedProfiles)}>
                <Text key='compare-action'/>
            </button>
        </Column>
        <Column grow={2}>
            <Text element="h4" key="compare-result" />
            {#if !displayedQuests && !displayedAchievements}
                <Text key="no-compare-runned" />
            {:else}
                {#snippet questTitle()} <h3> <Text key={"quests"} /> </h3> {/snippet}
                {#snippet achievementTitle()} <h3> <Text key={"achievements"} /> </h3> {/snippet}
                <Accordion
                    title={questTitle}
                    id={`quest-category-panel`}
                    forceOpen={false}
                >
                    <ul>
                        {#if displayedQuests}
                            {#each Object.entries(displayedQuests) as [questId, profileIds]}
                                <li>
                                    <Text key={`q${questId}`} name="name" />
                                    -
                                    {#each profileIds.map((id) => ownProfiles.profiles.find((p) => p.id === id)) as profile, index (profile?.id)}
                                        {#if profile}
                                            <ProfileChip
                                                name={profile.name}
                                                image={imageRoot + profilePictures[profile.image]}
                                            />
                                        {/if}
                                    {/each}
                                </li>
                            {/each}
                        {/if}
                    </ul>
                </Accordion>
                <Accordion
                    title={achievementTitle}
                    id={`achievements-category-panel`}
                    forceOpen={false}
                >
                    <ul>
                        {#if displayedAchievements}
                            {#each Object.entries(displayedAchievements) as [achievementId, profileIds]}
                                <li>
                                    <Text key={`a${achievementId}`} name="name" />
                                    -
                                    {#each profileIds.map((id) => ownProfiles.profiles.find((p) => p.id === id)) as profile, index (profile?.id)}
                                        {#if profile}
                                            <ProfileChip
                                                name={profile.name}
                                                image={imageRoot + profilePictures[profile.image]}
                                            />
                                        {/if}
                                    {/each}
                                </li>
                            {/each}
                        {/if}
                    </ul>
                </Accordion>
            {/if}
        </Column>
    </Row>
</Window>

<style>
    .profile {
        position: relative;
        display: flex;
        padding: 6px;
        border-radius: 4px;
        max-width: 260px;
        user-select: none;
        overflow: hidden;
        flex-grow: 1000;
        flex-shrink: 1;

        border-width: 1px;
        border-style: solid;

        transition:
            background-color 0.1s,
            border-color 0.1s;

        &.noclick {
            cursor: default !important;
        }

        &.selected {
            outline: 1px solid white;
            outline-offset: 1px;
            box-shadow: 0 0 7px rgba(255, 255, 255, 0.6);
        }

        & > .image-box {
            margin-right: 6px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            & > img {
                width: 32px;
                height: 32px;
            }
        }

        & .info {
            overflow: hidden;
            text-align: left;
            padding: 0 4px;
        }

        & .name {
            overflow: hidden;
            text-overflow: ellipsis;
        }
        & .title {
            overflow: hidden;
            text-overflow: ellipsis;
            opacity: 70%;
            font-style: italic;
            font-size: 0.8em;
        }
    }
</style>
