

Formation contains a ton of [utility classes](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/tree/master/packages/formation/sass/utilities "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/tree/master/packages/formation/sass/utilities") that are used extensively in vets-website. In order to get off of Formation and unblock the vets-website node upgrade we need to change the import source for these utilities in vets-website from Formation to css-library. This needs to be done with minimal disruption, and we need to ensure the new utilities continue to work the same way the old ones currently do.

## Considerations

- css-library relies on USWDS3 helpers to [generate utility classes](https://github.com/department-of-veterans-affairs/component-library/blob/ca594dabb0862defb44a2374b30cd2eab8957322/packages/css-library/src/utilities/background-color.scss#L1 "https://github.com/department-of-veterans-affairs/component-library/blob/ca594dabb0862defb44a2374b30cd2eab8957322/packages/css-library/src/utilities/background-color.scss#L1"). These are configured to generate utilities with the same names as their Formation equivalents.
    
- [Formation utilities](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/tree/master/packages/formation/sass/utilities "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/tree/master/packages/formation/sass/utilities") all make use of `!important` to override anything else in the style cascade
    
- All formation utilities are imported into [core.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/core.scss#L84-L101 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/core.scss#L84-L101"), which is then imported directly into vets-website in [style.scss](https://github.com/department-of-veterans-affairs/vets-website/blob/main/src/platform/site-wide/sass/style.scss#L5 "https://github.com/department-of-veterans-affairs/vets-website/blob/main/src/platform/site-wide/sass/style.scss#L5")
    
- css-library utilities currently get built into a [single utilities.css](https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/dist/stylesheets/utilities.css "https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/dist/stylesheets/utilities.css") file.
    
    - question: Is a single point of entry for all utilities the right way to go? Is there any sort of future state where individual apps should be able to import _only_ the utility stylesheets they need?
        
- Formation margin and padding utilities use a [$spacers](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/base/_b-variables.scss#L189 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/base/_b-variables.scss#L189") map that hard codes an 8px base. This is also used to provide a [$units](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/base/_b-variables.scss#L211 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/base/_b-variables.scss#L211") map that converts the 8px base values into `rem`. This is a challenge because it's predicated on the idea of a 10px root font size.
    
    - Example: 8px base = .8 value in `$units`, which computes to 8px. This can be observed on where `.vads-u-padding--1` computes to 8px padding on all sides.
        
    - This is a problem because now uses 16px as the root font-size, so unless we stick to pixel values, [which we currently don't](https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/dist/tokens/scss/variables.scss#L128 "https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/dist/tokens/scss/variables.scss#L128"), then all computed values will increase. Example: `$units-1` = 0.8rem = 12.8px computed.
        
    - Given the above, we may need to reevaluate how we're generating utilities using [spacing.json](https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/tokens/spacing.json "https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/tokens/spacing.json")
        
- At the beginning of 2024, we deprecated `vads-u-color--primary-darkest` in favor of `--primary-darker`. The original `--primary-darker` was shifted up to the new `--primary-dark`. There are still about 6 instances in vets-website where some utility with `--primary-darkest` is being used and 1 instance in content-build.
    
- It probably matters more when we attempt to shorten the names of all of our utilities, but there are 89 instances in vets-website where teams are targeting utility class names and overriding default styles with something custom in their application stylesheets. There are also instances where brand new custom css classnames are created, but adhere to the utility naming convention. Example: [these classes in form 2346](https://github.com/department-of-veterans-affairs/vets-website/blob/57e84fb2a3482662701f750cb8523b5d1bdaedf7/src/applications/disability-benefits/2346/sass/form-2346.scss#L34-L44 "https://github.com/department-of-veterans-affairs/vets-website/blob/57e84fb2a3482662701f750cb8523b5d1bdaedf7/src/applications/disability-benefits/2346/sass/form-2346.scss#L34-L44") do no actually exist in the design system.
    
- content-build uses utility classes extensively, but our current naming convention should keep things unchanged
    
- Formation [creates a breakpoints map](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/base/_b-breakpoints.scss#L56-L61 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/base/_b-breakpoints.scss#L56-L61") which is used to generate [responsive utility class names](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_display.scss#L22-L38 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_display.scss#L22-L38").
    
    - css-library has the same values in [breakpoints.json](https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/tokens/breakpoints.json "https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/tokens/breakpoints.json"), but these don't seem to be used in the [generated stylesheet](https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/dist/stylesheets/utilities.css "https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/dist/stylesheets/utilities.css"). Instead, the breakpoints it does use are listed in the [uswds-theme stylesheet](https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/src/stylesheets/_uswds-theme.scss#L17-L26 "https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/src/stylesheets/_uswds-theme.scss#L17-L26").
        
    - The actual uswds values are found [here](https://github.com/uswds/uswds/blob/62d297a7f09230e5fc4c8b9d9aaae099a4ebda2b/packages/uswds-core/src/styles/settings/_settings-utilities.scss#L33-L56 "https://github.com/uswds/uswds/blob/62d297a7f09230e5fc4c8b9d9aaae099a4ebda2b/packages/uswds-core/src/styles/settings/_settings-utilities.scss#L33-L56"). Current equivalent uswds-vads values:
        
        - mobile = xsmall-screen = 320px
            
        - mobile-lg = small-screen = 480px
            
        - tablet = tablet = 640px
            
        - desktop (1024px) = small-desktop-screen(1008px), 16px difference here
            
        - desktop-lg = large-screen = 1200px
            
        - Only thing that's really missing here is `medium-screen` equivalent of 768px, which is a challenge given how utilities are generated.
            
    - May need to investigate whether or not we can override and rename prefix values for responsive classes or add [custom settings to the uswds theme](https://designsystem.digital.gov/documentation/settings/#configuring-custom-uswds-settings-2 "https://designsystem.digital.gov/documentation/settings/#configuring-custom-uswds-settings-2").
        
    - css-library includes a [settings.scss](https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/src/utilities/settings.scss "https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/src/utilities/settings.scss") file which configures [USWDS3 settings](https://designsystem.digital.gov/documentation/settings/#utilities-settings-2 "https://designsystem.digital.gov/documentation/settings/#utilities-settings-2").
        
- Formation utilities that use breakpoints:
    
    - [border.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_border.scss#L35-L59 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_border.scss#L35-L59")
        
    - [display.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_display.scss#L22-L40 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_display.scss#L22-L40")
        
    - [flexbox.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_flexbox.scss#L43-L57 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_flexbox.scss#L43-L57") - multiple instances for each flex property
        
    - [font-size.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_font-size.scss#L26-L34 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_font-size.scss#L26-L34") - not sure we need this after our typography migration
        
    - [height-width.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_height-width.scss#L19-L27 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_height-width.scss#L19-L27") - multiple instances height, width, and min/max properties
        
    - [line-height.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_line-height.scss#L22-L30 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_line-height.scss#L22-L30") - not sure we need this after our typography migration
        
    - [margins.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_margins.scss#L83 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_margins.scss#L83")
        
    - [measure.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_measure.scss#L22 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_measure.scss#L22")
        
    - [padding.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_padding.scss#L36 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_padding.scss#L36")
        
    - [position.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_position.scss#L11 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_position.scss#L11")
        
    - [text-align.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_text-align.scss#L16 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_text-align.scss#L16")
        
    - [visibility.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_visibility.scss#L25 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/utilities/_visibility.scss#L25")
        

## Proposed Implementation

- We import css-library's `dist/src/stylesheets/utilities.css` into vets-website [style.scss](https://github.com/department-of-veterans-affairs/vets-website/blob/main/src/platform/site-wide/sass/style.scss#L6 "https://github.com/department-of-veterans-affairs/vets-website/blob/main/src/platform/site-wide/sass/style.scss#L6")
    
- We "turn off" utilities in formation's [core.scss](https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/core.scss#L84-L101 "https://github.com/department-of-veterans-affairs/veteran-facing-services-tools/blob/master/packages/formation/sass/core.scss#L84-L101") one by one, or two by two, whatever, by commenting out the import. Since we get all of the css-library utilities in one import, the ones that get turned off in Formation should automatically get inherited from css-libary, and we avoid the issue with all of the Formation utilities having `!important`.
    

- Example: comment out `@import 'utilities/background-color` in formation/core.scss, vets-website starts inheriting from css-library. They're named the same (for the most part, we'll need to update the ~6 instances of `color-primary-darkest`) so we shouldn't see any disruption.
    

- This allows us to not publish a ton of new versions of css library, and instead we're really just dealing with formation releases. It should keep us from porting over any of the formation utility code to css-library
    
- We start with the utilities that do not need the `medium-screen` breakpoint, which are: background-color, color, font-family, font-style, font-weight, and text-decoration
    

- background-color, and color will require some renaming in vets-website. When we worked on colors, we decided to stick with `color-primary-darker` and `-darkest` for the instances in vets-website. This is the time to update them.
    
- font-family may need some extra attention. The class names are shorter in css-library: `vads-u-font--sans` vs `vads-u-font-family--sans`, but I think the values are the same, need to confirm. 
    

- In addition to tickets for turning off formation imports 1 by 1, we need:
    

- a ticket for solving the `medium-screen` issue so we can also switch over the remaining utilities.
    
- a ticket for updating the rem values in [spacing.json](https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/tokens/spacing.json "https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/tokens/spacing.json") to account for our 16px root font-size
    
- a ticket for updating the rem values in [fonts.json](https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/tokens/fonts.json#L19-L25 "https://github.com/department-of-veterans-affairs/component-library/blob/main/packages/css-library/tokens/fonts.json#L19-L25") to account for our 16px root font-size
