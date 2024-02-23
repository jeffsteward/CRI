---
title: Dancing with Mirador
description: Animate the Mirador viewer
repo: 
layout: project
tags: 
dates: 2019-06-25
---

I hacked this together while learning about the actions dispatcher in <a href="https://projectmirador.org/">Mirador 3</a> during the Hands on Technical Workshop at the <a href="https://iiif.io/event/2019/goettingen/">2019 IIIF Conference</a> in Göttingen, Germany. The general thought was: "I want to make Mirador dance." 🤷

<div id="mirador" style="width: 100%;height: 500px; position: relative;"></div>
<script src="https://unpkg.com/mirador@latest/dist/mirador.min.js"></script>

<script type="text/javascript">
    const defaultSettings = {
    id: "mirador",
    // All of the settings (with descriptions (ﾉ^∇^)ﾉﾟ) located here:
    // https://github.com/ProjectMirador/mirador/blob/master/src/config/settings.js
    language: 'en', // The default language set in the application
    window: {
        allowClose: false, // Configure if windows can be closed or not
        allowFullscreen: false, // Configure to show a "fullscreen" button in the WindowTopBar
        allowMaximize: false, // Configure if windows can be maximized or not
        authNewWindowCenter: 'parent', // Configure how to center a new window created by the authentication flow. Options: parent, screen
        defaultSideBarPanel: 'info', // Configure which sidebar is selected by default. Options: info, attribution, canvas, annotations
        defaultView: 'single',  // Configure which viewing mode (e.g. single, book, gallery) for windows to be opened in
        hideAnnotationsPanel: true, // Configure to hide the annotations panel in the WindowSideBarButtons
        hideSearchPanel: true, // Configure to hide search panel in the WindowSideBarButtons
        hideWindowTitle: false, // Configure if the window title is shown in the window title bar or not
        sideBarOpenByDefault: true, // Configure if the sidebar (and its content panel) is open by default
    },
    workspace: {
        showZoomControls: false, // Configure if zoom controls should be displayed by default
        type: 'mosaic', // Which workspace type to load by default. Other possible values are "elastic"
    },
    workspaceControlPanel: {
        enabled: false, // Configure if the control panel should be rendered.  Useful if you want to lock the viewer down to only the configured manifests
    },
    themes: {
        bold: {
            palette: {
                type: 'light',
                primary: {
                    main: '#ff00ff'
                },
                secondary: {
                    main: "#00ff00"
                }
            }
        },
    },
    selectedTheme: 'bold',
    };

    // Fire up an instance of Mirador Viewer
    const mirador = Mirador.viewer(defaultSettings);

    let currentLanguage = 0;
    let languages = Object.keys(mirador.store.getState().config.availableLanguages);
    // Remove the default language so we don't see it twice during the animation
    languages.splice(languages.indexOf(defaultSettings.language),1);

    let manifestIds = [
    "https://iiif.harvardartmuseums.org/manifests/object/6772",
    "https://iiif.harvardartmuseums.org/manifests/object/230119",
    "https://iiif.harvardartmuseums.org/manifests/object/168614",
    ];
    currentManifestIndex = 0;

    let currentIntervalIndex;

    function changeLanguage() {
        mirador.store.dispatch(
            Mirador.actions.updateConfig({
                language: languages[currentLanguage]
            })
        );
        currentLanguage +=1;
        if (currentLanguage >= languages.length) {
            currentLanguage = 0;
            clearInterval(currentIntervalIndex);
            currentIntervalIndex = setInterval(makeWindow, 2000);
        }
    }

    function makeWindow() {
        mirador.store.dispatch(
            Mirador.actions.addWindow({
                manifestId: manifestIds[currentManifestIndex],
            })
    );         

    currentManifestIndex += 1;
    if (currentManifestIndex >= manifestIds.length) {
        clearInterval(currentIntervalIndex);
        currentIntervalIndex = setInterval(changeLayout, 1000);
    }
    }

    function changeLayout() {
        mirador.store.dispatch(
                Mirador.actions.updateConfig({
                    workspace: {
                        type: 'elastic'
                    }
                })
        );
        
        clearInterval(currentIntervalIndex);
        arrangeWindows();
    }

    function changeTheme() {
        mirador.store.dispatch(
            Mirador.actions.updateConfig({
            selectedTheme: 'light'
            })
        );
    }

    function arrangeWindows() {
        let windows = Object.keys(mirador.store.getState().elasticLayout);

        windows.forEach(w => {
            let xPos = Math.round(Math.floor(Math.random()*((window.innerWidth-200)-0+1)+0));
            let yPos = Math.round(Math.floor(Math.random()*((window.innerHeight-200)-0+1)+0));

            mirador.store.dispatch(
                Mirador.actions.updateElasticWindowLayout(w, {x:xPos, y:yPos})
            );            
        });

        currentIntervalIndex = setInterval(deleteWindows, 2000);
    }

    function deleteWindows() {
        let windows = Object.keys(mirador.store.getState().elasticLayout);

        if (windows.length > 0) {
            let w = mirador.store.getState().elasticLayout[windows[0]];
            mirador.store.dispatch(
                Mirador.actions.removeWindow(w.windowId)
            );
        } else {
            clearInterval(currentIntervalIndex);
            restoreUi();
        }
    }

    function restoreUi() {
        mirador.store.dispatch(
            Mirador.actions.updateConfig({
                language: 'en',
                window: {
                    allowClose: true, // Configure if windows can be closed or not
                    allowFullscreen: true, // Configure to show a "fullscreen" button in the WindowTopBar
                    allowMaximize: true, // Configure if windows can be maximized or not
                    defaultView: 'single',  // Configure which viewing mode (e.g. single, book, gallery) for windows to be opened in
                    hideAnnotationsPanel: false, // Configure to hide the annotations panel in the WindowSideBarButtons
                    sideBarOpenByDefault: false, // Configure if the sidebar (and its content panel) is open by default
                },
                workspaceControlPanel: {
                    enabled: true, // Configure if the control panel should be rendered.  Useful if you want to lock the viewer down to only the configured manifests
                },
                workspace: {
                        type: 'mosaic'
                    }
            })
        );
    }

    currentIntervalIndex = setInterval(changeLanguage, 1000);
</script>