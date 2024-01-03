<script>
    import { onMount } from "svelte";
    import { useParams } from "svelte-navigator";
    import { writable } from "svelte/store";
    import { Link } from "svelte-navigator";

    let project = writable(null);

    const { subscribe } = useParams();

    let certification_steps = [];
    let purpose_of_survey_steps = [];


    //Гет запрос на сервер при переходе на страницу
    // Fetch project details when the component is mounted
    onMount(() => {
        const unsubscribe = subscribe((params) => {
        const { project_id } = params;

        fetchProjectDetails(project_id);
        });

        return unsubscribe;
    });

    async function fetchProjectDetails(project_id) {
        try {
        const response = await fetch(`http://127.0.0.1:5000/EditProject/${project_id}`, {
            method: "GET",
            headers: {
            "Content-Type": "application/json",
            },
        });

        if (response.ok) {
            const data = await response.json();
            if (data.status === "success") {
            // Convert ObjectId to string in the project data
            data.project._id = String(data.project._id);
            project.set(data.project);
            } else {
            console.error("Failed to fetch project details.");
            }
        } else {
            console.error("Failed to fetch project details.");
        }
        } catch (error) {
        console.error("Error during fetch:", error);
        }
    }


    //Добавления шага
    $: {
        if ($project) {
            certification_steps = $project.certification_steps || [];
        }
    }

    // ... (ваш существующий код)

    // Обновленная функция addStep, которая добавляет шаг и обновляет соответствующий подраздел
    async function addStep(event) {
        event.preventDefault();

        const stepDescription = document.getElementById("step_description").value;
        const currentSection = document.getElementById("current_section").value;

        try {
            const response = await fetch(`http://127.0.0.1:5000/edit_project/${$project._id}/add_step`, {
                method: "POST",
                headers: {
                    "Content-Type": "application/x-www-form-urlencoded",
                },
                body: `step_description=${encodeURIComponent(stepDescription)}&section=${encodeURIComponent(currentSection)}`,
            });

            if (response.ok) {
                const data = await response.json();
                if (data.status === "success") {
                    // Обновляем certification_steps после успешного добавления шага
                    certification_steps = data.updated_certification_steps;
                    // Очищаем поле ввода после успешного добавления
                    document.getElementById("step_description").value = '';
                } else {
                    console.error("Failed to add step:", data.message);
                }
            } else {
                console.error("Failed to add step:", response.statusText);
            }
        } catch (error) {
            console.error("Error during add step:", error);
        }
    }

    //Удаление шага
    async function deleteStep(section, step) {
        try {
            const response = await fetch(`http://127.0.0.1:5000/edit_project/${$project._id}/delete_step`, {
                method: "POST",
                headers: {
                    "Content-Type": "application/x-www-form-urlencoded",
                },
                body: `step_to_delete=${encodeURIComponent(step)}&section=${encodeURIComponent(section)}`,
            });

            if (response.ok) {
                const data = await response.json();
                if (data.status === "success") {
                    // Обновите certification_steps после успешного удаления шага
                    certification_steps = data.updated_certification_steps;
                } else {
                    console.error("Failed to delete step:", data.message);
                }
            } else {
                console.error("Failed to delete step:", response.statusText);
            }
        } catch (error) {
            console.error("Error during delete step:", error);
        }
    }

    /////////////////////////////////////////       Open close section     /////////////////////////////////////////

    var currentOpenSubsection = null;
    var currentOpenSubsectionButton = null;

function showSubsection(subsection) {
    var subsectionElement = document.getElementById(subsection + "-section");
    var subsectionButton = document.getElementById(subsection + "-button");

    // Если текущий подраздел открыт, смените его цвет обратно
    if (currentOpenSubsectionButton && currentOpenSubsectionButton !== subsectionButton) {
        currentOpenSubsectionButton.classList.remove("button-clicked");
    }

    // Переключите класс кнопки подраздела
    subsectionButton.classList.toggle("button-clicked");

    // Сохраните текущую кнопку подраздела
    currentOpenSubsectionButton = subsectionButton;

    if (currentOpenSubsection === subsection) {
        // If the same sub-section is clicked again, close it
        subsectionElement.style.display = "none";
        currentOpenSubsection = null;
    } else {
        // Close the previously open sub-section, if any
        if (currentOpenSubsection) {
            document.getElementById(currentOpenSubsection + "-section").style.display = "none";
        }

        // Close other subsections with the class "infomation"
        var allSubsections = document.querySelectorAll(".infomation");
        allSubsections.forEach(function (subsection) {
            if (subsection.id !== subsection + "-section") {
                subsection.style.display = "none";
            }
        });

        // Open the clicked sub-section
        subsectionElement.style.display = "block";
        currentOpenSubsection = subsection;
    }

    // Show the "Add Step" form and set the current section
    var addStepFormContainer = document.getElementById("add-step-form-container");
    addStepFormContainer.style.display = "block";
    var currentSectionField = document.getElementById("current_section");
    currentSectionField.value = subsection;
}

    function resetButtonColor(sectionName) {
    // Найти все кнопки с классом "button-clicked" и удалить этот класс
    var clickedButtons = document.querySelectorAll('.button-clicked');
    clickedButtons.forEach(function (button) {
        if (button.id !== sectionName + "-button") {
            button.classList.remove('button-clicked');
        }
    });
}

function resetSubsectionButtonColor() {
    // Найти все кнопки с классом "button-clicked" и удалить этот класс
    var clickedButtons = document.querySelectorAll('.subsection-button-clicked');
    clickedButtons.forEach(function (button) {
        button.classList.remove('subsection-button-clicked');
    });
}

function closeOtherSections(exceptSectionId) {
    var sections = document.querySelectorAll(".osnova");
    sections.forEach(function (section) {
        if (section.id !== exceptSectionId) {
            section.style.display = "none";
        }
    });
}

function hideAddStepForm() {
    var addStepFormContainer = document.getElementById("add-step-form-container");
    addStepFormContainer.style.display = "none";
}

function closeSubsections(sectionId) {
    var section = document.getElementById(sectionId);
    if (section) {
        var subsections = section.getElementsByClassName("infomation"); // Replace "infomation" with the class name of your subsections
        for (var i = 0; i < subsections.length; i++) {
            subsections[i].style.display = "none";
        }
    }
}


function showIntroductionSections() {
    // Закройте другие разделы
    closeOtherSections("introduction-sections");
    closeSubsections("introduction-sections");
    hideAddStepForm();

    // Сбросьте цвет предыдущей кнопки
    resetButtonColor();

    // Получите элемент кнопки
    var introductionSectionsButton = document.getElementById("introduction-sections-button");

    // Переключите класс кнопки
    introductionSectionsButton.classList.toggle("button-clicked");

    // Получите элемент секции
    var introductionSections = document.getElementById("introduction-sections");

    // Переключите отображение секции
    if (introductionSections.style.display === "none") {
        introductionSections.style.display = "block";
    } else {
        introductionSections.style.display = "none";
    }
}

function showElectricalSystemsSections() {
    closeOtherSections("electrical-systems-sections");
    closeSubsections("electrical-systems-sections");
    hideAddStepForm();

    // Сбросьте цвет предыдущей кнопки
    resetButtonColor();

    // Получите элемент кнопки
    var electricalSystemsButton = document.getElementById("electrical-systems-button");

    // Переключите класс кнопки
    electricalSystemsButton.classList.toggle("button-clicked");

    // Получите элемент секции
    var electricalSystemsSections = document.getElementById("electrical-systems-sections");

    // Переключите отображение секции
    if (electricalSystemsSections.style.display === "none") {
        electricalSystemsSections.style.display = "block";
    } else {
        electricalSystemsSections.style.display = "none";
    }
}



//HullSections
function showHullSections() {
    // Закройте другие разделы
    closeOtherSections("hull-sections");
    closeSubsections("hull-sections");
    hideAddStepForm();

    // Сбросьте цвет предыдущей кнопки
    resetButtonColor();

    // Получите элемент кнопки
    var hullSectionsButton = document.getElementById("hull-sections-button");

    // Переключите класс кнопки
    hullSectionsButton.classList.toggle("button-clicked");

    // Получите элемент секции
    var hullSections = document.getElementById("hull-sections");

    // Переключите отображение секции
    if (hullSections.style.display === "none") {
        hullSections.style.display = "block";
    } else {
        hullSections.style.display = "none";
    }
}


function showAboveSections() {
    // Закройте другие разделы
    closeOtherSections("above-sections");
    closeSubsections("above-sections");
    hideAddStepForm();

    // Сбросьте цвет предыдущей кнопки
    resetButtonColor();

    // Получите элемент кнопки
    var aboveSectionsButton = document.getElementById("above-sections-button");

    // Переключите класс кнопки
    aboveSectionsButton.classList.toggle("button-clicked");

    // Получите элемент секции
    var aboveSections = document.getElementById("above-sections");

    // Переключите отображение секции
    if (aboveSections.style.display === "none") {
        aboveSections.style.display = "block";
    } else {
        aboveSections.style.display = "none";
    }
}

// Повторите тот же шаблон для других функций
function showBelowSections() {
    closeOtherSections("below-sections");
    closeSubsections("below-sections");
    hideAddStepForm();

    resetButtonColor();

    var belowSectionsButton = document.getElementById("below-sections-button");

    belowSectionsButton.classList.toggle("button-clicked");

    var belowSections = document.getElementById("below-sections");

    if (belowSections.style.display === "none") {
        belowSections.style.display = "block";
    } else {
        belowSections.style.display = "none";
    }
}

// Повторите тот же шаблон для других функций
function showCathodicProtectionSections() {
    closeOtherSections("cathodic-protection-sections");
    closeSubsections("cathodic-protection-sections");
    hideAddStepForm();

    resetButtonColor();

    var cathodicProtectionSectionsButton = document.getElementById("cathodic-protection-sections-button");

    cathodicProtectionSectionsButton.classList.toggle("button-clicked");

    var cathodicProtectionSections = document.getElementById("cathodic-protection-sections");

    if (cathodicProtectionSections.style.display === "none") {
        cathodicProtectionSections.style.display = "block";
    } else {
        cathodicProtectionSections.style.display = "none";
    }
}

//Helm Sections
function showHelmStationSections() {
    closeOtherSections("helm-station-sections");
    closeSubsections("helm-station-sections");
    hideAddStepForm();
    resetButtonColor();

    var helmStationSectionsButton = document.getElementById("helm-station-sections-button");

    helmStationSectionsButton.classList.toggle("button-clicked");

    var helmStationSections = document.getElementById("helm-station-sections");

    if (helmStationSections.style.display === "none") {
        helmStationSections.style.display = "block";
    } else {
        helmStationSections.style.display = "none";
    }
}

function showCabinInteriorSections() {
    closeOtherSections("cabin-interior-sections");
    closeSubsections("cabin-interior-sections");
    hideAddStepForm();
    resetButtonColor();

    var cabinInteriorSectionsButton = document.getElementById("cabin-interior-sections-button");

    cabinInteriorSectionsButton.classList.toggle("button-clicked");

    var cabinInteriorSections = document.getElementById("cabin-interior-sections");

    if (cabinInteriorSections.style.display === "none") {
        cabinInteriorSections.style.display = "block";
    } else {
        cabinInteriorSections.style.display = "none";
    }
}

function showInboardPropulsionSections() {
    closeOtherSections("inboard-propulsion-sections");
    closeSubsections("inboard-propulsion-sections");
    hideAddStepForm();
    resetButtonColor();

    var inboardPropulsionSectionsButton = document.getElementById("inboard-propulsion-sections-button");

    inboardPropulsionSectionsButton.classList.toggle("button-clicked");

    var inboardPropulsionSections = document.getElementById("inboard-propulsion-sections");

    if (inboardPropulsionSections.style.display === "none") {
        inboardPropulsionSections.style.display = "block";
    } else {
        inboardPropulsionSections.style.display = "none";
    }
}

function showSteeringSystemSections() {
    closeOtherSections("steering-system-sections");
    closeSubsections("steering-system-sections");
    hideAddStepForm();
    resetButtonColor();

    var steeringSystemSectionsButton = document.getElementById("steering-system-sections-button");

    steeringSystemSectionsButton.classList.toggle("button-clicked");

    var steeringSystemSections = document.getElementById("steering-system-sections");

    if (steeringSystemSections.style.display === "none") {
        steeringSystemSections.style.display = "block";
    } else {
        steeringSystemSections.style.display = "none";
    }
}


function showTankageSections() {
    closeOtherSections("tankage-sections");
    closeSubsections("tankage-sections");
    hideAddStepForm();
    resetButtonColor();

    var tankageSectionsButton = document.getElementById("tankage-sections-button");

    tankageSectionsButton.classList.toggle("button-clicked");

    var tankageSections = document.getElementById("tankage-sections");

    if (tankageSections.style.display === "none") {
        tankageSections.style.display = "block";
    } else {
        tankageSections.style.display = "none";
    }
}

function showSafetyEquipmentSections() {
    closeOtherSections("safety-equipment-sections");
    closeSubsections("safety-equipment-sections");
    hideAddStepForm();
    resetButtonColor();

    var safetyEquipmentSectionsButton = document.getElementById("safety-equipment-sections-button");

    safetyEquipmentSectionsButton.classList.toggle("button-clicked");

    var safetyEquipmentSections = document.getElementById("safety-equipment-sections");

    if (safetyEquipmentSections.style.display === "none") {
        safetyEquipmentSections.style.display = "block";
    } else {
        safetyEquipmentSections.style.display = "none";
    }
}

</script>

<style>
    .top {
    display: flex;
    justify-content: space-evenly;
    margin: 15px;
    width: 92%;
}

</style>



<main>
    {#if $project}
    <h1 class="sitename">Survzilla</h1>
    <h1>{$project.first_name}</h1>
    <Link to="/glav" class="btn btn-danger">Выйти из проекта</Link><!-- кнопка перехода на страницу создания проектов -->
    <div class="top">
        <h1>Edit Project: { $project.first_name } { $project.last_name } vessel name: { $project.vessel_name }</h1>
    </div>
    {/if}
    <div class="deck">
        <button id="introduction-sections-button" on:click={showIntroductionSections}>INTRODUCTION</button>
        <button id="hull-sections-button" on:click={showHullSections}>Hull, DECK & SUPERSTRUCTURE</button>
        <button id="above-sections-button" on:click={showAboveSections}>ABOVE</button>
        <button id="below-sections-button" on:click={showBelowSections}>BELOW</button>
        <button id="cathodic-protection-sections-button"  on:click={showCathodicProtectionSections}>CATHODIC PROTECTION</button>
        <button id="helm-station-sections-button" on:click={showHelmStationSections}>HELM STATION & NAVIGATIONAL</button>
        <button id="cabin-interior-sections-button" on:click={showCabinInteriorSections}>CABIN INTERIOR APPOINTMENTS</button>
        <button id="electrical-systems-button" on:click={showElectricalSystemsSections}>ELECTRICAL SYSTEMS</button>
        <button id="inboard-propulsion-sections-button" on:click={showInboardPropulsionSections}>INBOARD PROPULSION SYSTEM</button>
        <button id="steering-system-sections-button" on:click={showSteeringSystemSections}>STEERING SYSTEM</button>
        <button id="tankage-sections-button" on:click={showTankageSections}>TANKAGE</button>
        <button id="safety-equipment-sections-button" on:click={showSafetyEquipmentSections}>SAFETY EQUIPMENT</button>
    </div>
    <div class="osnova" style="display: none;" id="introduction-sections">
        <h1 class="forosnova">Introduction sections</h1>
        <button id="certification-button" on:click={() => showSubsection('certification')}>CERTIFICATION</button>
        <button id="purpose_of_survey-button" on:click={() => showSubsection('purpose_of_survey')}>PURPOSE OF SURVEY</button>
    </div>
    <div class="osnova" style="display: none;" id="hull-sections">
        <h1 class="forosnova">Hull sections</h1>

    </div>
    <div class="osnova" style="display: none;" id="above-sections">
        <h1 class="forosnova">Above sections</h1>

    </div>
    <div class="osnova" style="display: none;" id="below-sections">
        <h1 class="forosnova">Below sections</h1>

    </div>
    <div class="osnova" style="display: none;" id="cathodic-protection-sections">
        <h1 class="forosnova">Cathodic protection sections</h1>

    </div>
    <div class="osnova" style="display: none;" id="helm-station-sections">
        <h1 class="forosnova">Helm station sections</h1>

    </div>
    <div class="osnova" style="display: none;" id="cabin-interior-sections">
        <h1 class="forosnova">Cabin interior sections</h1>

    </div>
    
    <div class="osnova" style="display: none;" id="electrical-systems-sections">
        <h1 class="forosnova">Electrical systems sections</h1>

    </div>
    
    <div class="osnova" style="display: none;" id="inboard-propulsion-sections">
        <h1 class="forosnova">Inboard propulsion sections</h1>

    </div>
    
    <div class="osnova" style="display: none;" id="steering-system-sections">
        <h1 class="forosnova">Steering system sections</h1>

    </div>
    
    <div class="osnova" style="display: none;" id="tankage-sections">
        <h1 class="forosnova">Tankage sections</h1>

    </div>
    
    <div class="osnova" style="display: none;" id="safety-equipment-sections">
        <h1 class="forosnova">Safety equipment sections</h1>

    </div>
    <!--Добавление шага-->
    <div id="add-step-form-container" style="display: none;">
        <h2>Add Step:</h2>
        <form on:submit={addStep}><!--<form method="POST" action="/edit_project/{{ project_id }}/add_step">-->
            <label for="step_description">Step Description:</label>
            <input type="text" id="step_description" name="step_description">
            
            <input type="hidden" name="section" id="current_section" value="">
            <button type="submit">Add Step</button>
        </form>
    </div>

    <!--вывод и удаление шага-->
    <!--certification -->
    <div id="certification-section" style="display: none;" class="infomation">
        <h2>certification:</h2>
        {#if certification_steps && certification_steps.length > 0}
            <ul>
                {#each certification_steps as step}
                    <li>{step}</li>
                    <form on:submit|preventDefault={() => deleteStep('certification', step)}>
                        <input type="hidden" name="step_to_delete" value={step} />
                        <input type="hidden" name="section" value="certification" />
                        <button type="submit">Delete</button>
                    </form>
                {/each}
            </ul>
        {:else}
            <p>No steps added yet.</p>
        {/if}
    </div>
    <!-- purpose_of_survey-section -->
    <div id="purpose_of_survey-section" style="display: none;" class="infomation">
        <h2>purpose of survey:</h2>
        {#if purpose_of_survey_steps && purpose_of_survey_steps.length > 0}
            <ul>
                {#each purpose_of_survey_steps as step}
                    <li>{step}</li>
                    <form on:submit|preventDefault={() => deleteStep('purpose_of_survey', step)}>
                        <input type="hidden" name="step_to_delete" value={step} />
                        <input type="hidden" name="section" value="purpose_of_survey" />
                        <button type="submit">Delete</button>
                    </form>
                {/each}
            </ul>
        {:else}
            <p>No steps added yet.</p>
        {/if}
    </div>
</main>