## Terminology
- Formation: The current base of the VADS
- VADS: Veteran Affairs Design System
- USWDS#: United States Web Design System(1/3)
- [css-library](https://github.com/department-of-veterans-affairs/component-library/tree/main/packages/css-library): The new base of VADS using USWDS3
## Background
VA.gov is currently using [Formation](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/tree/master/packages/formation) as the base for its design system. The issue here is that Formation is running off of USWDS1 which is outdated, as USWDS is currently on version 3.  

## Challenges
- How do we safely cut over from Formation to USWDS3 while breaking as little as possible?
- How do we migrate while relying on application teams to update app code as little as possible? 
## Considerations
1. Formation has root typography, spacing, and other CSS settings that differ from USWDS3. For example, the root font size in Formation is 10px versus 16.92px in USWDS3.
2. We want to avoid changing Formation as much as possible so we have it to fall back on if migration goes awry, but there may be instances where it's unavoidable. For example, [va.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/214b7f41b8be70310820013c1a6bf3020897e349/packages/formation/sass/base/_va.scss#L1)Directly targets elements with css element selectors. In order to migrate to something new, we may need to remove these. 
3. vets-website depends on uswds 1.6.10 in its [packages.json](https://github.com/department-of-veterans-affairs/vets-website/blob/1229bcfcd559435bcce3d23c8bf899bbb6650bd3/package.json#L313)
4. css-library currently lives as a package in component-library. This isn't representative of what css-library is. It should be moved into its own directory its releases should be managed independently of the rest of the component library

## Migration Strategy
This strategy proposes a multi-tiered approach to migration that builds on the discovery work Brooks conducted in Q4 2022. Specifically, this approach makes use of [minimal.scss](https://github.com/department-of-veterans-affairs/vets-website/blob/d7663c0922a0fc850ec2f8370e1f66da2107385c/src/platform/site-wide/sass/minimal.scss#L0-L1) and the ability to import it into individual applications, opting out of the sitewide stylesheet. In this approach, we could work with the check-in application which already uses `minimal.scss`, or import `minimal.scss` in one of our playground applications. 

Using scss utilities as an example, high level migration looks like this:

1. Identify utility file to migrate out of Formation
2. Create a corresponding utility file in css-library
3. Swap the import in `minimal.scss` so we can test in isolation for parity and regression
4. Once testing has completed, promote the import to [the sitewide stylesheet](https://github.com/department-of-veterans-affairs/vets-website/blob/20cfa222cbd98867014f5f4cc39433b59a06f3fa/src/platform/site-wide/sass/style.scss#L1) and remove it from Formation's [core.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/a27df6e71e127c48b91669a94a99e9edda371af2/packages/formation/sass/core.scss#L67) where it gets brought into vets-website.
5. Once migration has been completed and verified to work, we can address optimizations around class naming and file size discussed in [this pull request](https://github.com/department-of-veterans-affairs/component-library/pull/452)  

Below are some specific steps that help us achieve our migration path outlined above:
#### 1. Move css-library out of component-library/packages
- css-library gets published any time the rest of the component library does because of [this github action](https://github.com/department-of-veterans-affairs/component-library/blob/main/.github/workflows/publish.yml#L37-L39). css-library is really a distinct product from the component-library and should be maintained separately. Some options are:
	- Move css-library into something like `component-library/libraries/` and update how publishing works. This would mean updates to our github action and package.json such that `workspaces` includes `libraries`. 
		- Thought: `component-library` would no longer be an accurate name for this repository. `va-design-system` may be more descriptive of what the repo contains, if css-library stays in it. 
	- Move css-library into its own repository where it's maintained separately

#### 2. Migrate global base styles from Formation to css-library
- Font size
	- **Proposal**: Move font-size declaration into css-library's [elements.scss](https://github.com/department-of-veterans-affairs/component-library/blob/8f90c0c97c5ebf4aead0467f6530f9d7e4a6da84/packages/css-library/src/stylesheets/elements.scss#L3)
		- Formation specifies font-size on both the [html and body element selectors](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/214b7f41b8be70310820013c1a6bf3020897e349/packages/formation/sass/base/_va.scss#L1-L22) 
		-  Temporarily override `html` and `body` element selectors in `va.scss` by using `!important` in `elements.scss` so we can test in isolation using `minimal.scss`
		- Once css-library is verified to work, remove font-size from the html and body element selectors in Formation, promote `elements.scss` to the vets-website sitewide stylesheet, and remove `!important` from the style declarations. 
- Color
	- **Proposal**: Update variable values in Formation to use USWDS3 hex values
		- There are 128 occurrences of some variation of `$color-primary-` in Formation, and 193 occurrences of `$color-primary-` variations in vets-website. Until `va.scss` and Formation utilities are deprecated, front loading this change directly in Formation allows users to acclimate to one of the most noticeable visual changes in our migration. 
		- Once deprecation of `va.scss` and Formation utilities has been complete we will sunset [b-variables.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/97382cc5d7f8327d2c1ebfb89b3c472cc222c30b/packages/formation/sass/base/_b-variables.scss#L39) and cut over to css-library's [variables.scss](https://github.com/department-of-veterans-affairs/component-library/blob/bef641ef2c85077d4a013d5fe32030fd176912bc/packages/css-library/dist/tokens/scss/variables.scss#L5). This will be imported into the vets-website site-wide stylesheet. Because variable names (`$color-primary-`) have remained the same wherever possible, there should be no visual regressions, as color hex values were front-loaded in Formation

#### 3. Migrate utility classes from Formation to css-library 
**Proposal**: Keep existing Formation class naming conventions in css-library and temporarily use `!important` to override Formation utilities.
	- Almost all Formation utilities use `!important` to enforce style overrides. 
	- [Previous research](https://github.com/department-of-veterans-affairs/vets-design-system-documentation/issues/860#issuecomment-1170609546) by Brooks found that additional styling was necessary to maintain visual parity with some utility classes
	- css-library should generate the same names Formation uses
	- Once the utility is generated, we will replace the Formation utility import in minimal.scss with the one from css-library to test in isolation
	- After testing has passed, the utility will be promoted to the site wide style sheet and the corresponding import should be removed from Formation's [core.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/a27df6e71e127c48b91669a94a99e9edda371af2/packages/formation/sass/core.scss#L68)
	- After the Formation `core.scss` utility import has been removed, `!important` can be removed from the css-library utility. 

- Use `render-utilities-in()` to generate utility classes that leverage USWDS3 tokens. A lot of the work for utility migration seems to already be complete. Existing css-library utility style sheets can be found [here](https://github.com/department-of-veterans-affairs/component-library/tree/main/packages/css-library/src/utilities). We need to verify the work done is usable. 
- These existing utilities already make use of token values and generate utility classes usable in vets-website. 

###### Borders as an example

border.scss:
```scss
@use 'uswds-core/src/styles/functions' as *;
@use '../tokens/border' as *;
@use '../tokens/color' as *;
@use './settings' as *;

$u-border-base: (
 border: (
  base: 'vads-border',
  modifiers: (
   noModifier: '',
   'top': '-top',
   'right': '-right',
   'bottom': '-bottom',
   'left': '-left',
 ),
  values: map-collect($tokens-border),
  settings: $border-settings,
  property: 'border',
  type: 'utility',
 ),
);
```

`base`: 
	- type: string 
	- Base class name
`modifiers`: 
	- type: scss map 
	- Suffixes that get appended to the base
`values`: 
	- type: scss map 
	- uses tokens
	- Values are appended to modifiers
`settings`:
	- type: scss map
	- uswds settings passed in that will output additional scss rules
`property`:
	- type: string
	- The primitive css property being targeted
`type`:
	- type: string
	- uswds definition (???)

`$u-border-base` is passed into the USWDS3 provided [render-utilities-in function](https://github.com/uswds/uswds/blob/develop/packages/uswds-core/src/styles/mixins/_utility-builder.scss#L353): 
```scss
@use 'uswds-core/src/styles/mixins/utility-builder' as *;
@use '../utilities' as *;

@include render-utilities-in($u-border-base)
```

Which outputs these utility classes:
```css
.vads-border-0 {
 border: 0;
}
.vads-border-top-0 {
 border-top: 0;
}
.vads-border-right-0 {
 border-right: 0;
}
.vads-border-bottom-0 {
 border-bottom: 0;
}
.vads-border-left-0 {
 border-left: 0;
}
.vads-border-1px {
 border: 1px solid;
}
/* generating the above for all token values specified in $tokens-border*/
```

These utility classes importable into vets-website. 

#### 4. Post Migration: Refactor utility class names
Once utilities have been fully sunsetted in Formation, we can confidently begin refactoring class names to take advantage of [smaller file sizes](https://github.com/department-of-veterans-affairs/component-library/pull/452#issue-1258979175), and perform any other optimizations that are surfaced during the migration process. 

### Advantages of this approach
- The design system team is less dependent on application teams to migrate their individual applications
- App by App imports become less of a worry to maintain (less PRs into content-build for local style opt in, less PRs into vets-website so apps can use minimal.scss)
- Site-wide migration to USWDS3 styles happens earlier because we actively promote utilities to the site wide style sheet
- Roll back should be relatively simple in vets-website if something goes off the rails in production, as we could just roll back the version numbers for Formation and css-library

### Disadvantages of this approach
- Temporary use of `!important` to override Formation utility classes
- More pull request coordination when deprecating Formation utilities:
	- PRs to update Formation version numbers when core.scss imports are removed
	- PRs to update css-library version numbers when new utilities roll out
	- PRs in vets-website to update both of those package version numbers 
- Requires active modification of Formation

