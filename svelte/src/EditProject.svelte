<script>
    import { onMount } from "svelte";
    import { useParams } from "svelte-navigator";
    import { writable } from "svelte/store";
    import { Link } from "svelte-navigator";

    let project = writable(null);
    

    const { subscribe } = useParams();

    let navigational_lights_steps = [];


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


    //Post запрос для допавлении шага
    let currentSection = "gen_info";

    function addStep(section) {
        currentSection = section; // Update the current section

        // Assuming you have the step_description and image values set in your component
        fetch(`/edit_project/${project_id}/add_step`, {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify({
            step_description,
            image, // Add the image value if needed
            section,
        }),
        })
        .then((response) => {
            if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
            }
            return response.json();
        })
        .then((data) => {
            console.log(data); // Handle the response from the server
        })
        .catch((error) => {
            console.error("Fetch error:", error);
        });
    }

    


    (document).ready(function () {
    // Обработчик для формы добавления подраздела
    ("#add-subsection-form").submit(function (event) {
        event.preventDefault();
        var subsectionName = ("#subsection_name").val();
        if (subsectionName) {
            // Отправить форму, только если имя подраздела не пустое
            (this).unbind("submit").submit();
        } else {
            alert("Введите имя подраздела.");
        }
    });

    // Обработчик для формы добавления ячейки
    ("#add-cell-form").submit(function (event) {
        event.preventDefault();
        var cellName = ("#cell_name").val();
        var cellDescription = ("#cell_description").val();
        if (cellName && cellDescription) {
            // Отправить форму, только если имя ячейки и описание не пустые
            (this).unbind("submit").submit();
        } else {
            alert("Введите имя ячейки и описание.");
        }
    });
});




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

//Customer section show close
function showCreateSectionModal() {
    var modal = document.getElementById("createSectionModal");
    modal.style.display = "block";
}

function closeCreateSectionModal() {
    var modal = document.getElementById("createSectionModal");
    modal.style.display = "none";
}


function showSubsections(sectionName) {
    closeAllSections();
    var subsectionsDiv = document.getElementById("subsections-" + sectionName);

    // Сбросьте цвет предыдущей кнопки
    resetButtonColor(sectionName);
    

    if (subsectionsDiv.style.display === "none") {
        subsectionsDiv.style.display = "block";
    } else {
        subsectionsDiv.style.display = "none";
    }

    // Получите элемент кнопки
    var sectionButton = document.getElementById(sectionName + "-button");

    // Переключите класс кнопки
    sectionButton.classList.toggle("button-clicked");
}




function closeAllSections() {
    var allSections = document.querySelectorAll('.osnova');
    allSections.forEach(function (section) {
        section.style.display = "none";
    });

    // Сбросьте цвет всех кнопок
    var allButtons = document.querySelectorAll('.button-clicked');
    allButtons.forEach(function (button) {
        button.classList.remove('button-clicked');
    });
}



//Customer subsection show close
function showCreateSubsectionModal(sectionName) {
    var modal = document.getElementById("createSubsectionModal");
    var input = modal.querySelector("#section_name_input");

    input.value = sectionName;

    modal.style.display = "block";
}

function closeCreateSubsectionModal() {
    var modal = document.getElementById("createSubsectionModal");
    modal.style.display = "none";
}



//Customer cells show close
function showCreateCellModal(sectionName, subsectionName) {
    var modal = document.getElementById("createCellModal");
    var sectionInput = modal.querySelector("#section_name_input");
    var subsectionInput = modal.querySelector("#subsection_name_input");

    sectionInput.value = sectionName;
    subsectionInput.value = subsectionName;

    modal.style.display = "block";
}

function closeCreateCellModal() {
    var modal = document.getElementById("createCellModal");
    modal.style.display = "none";
}
//
function showCells(sectionName, subsectionName) {
    var cellsDiv = document.getElementById("cells-" + sectionName + "-" + subsectionName);

    if (cellsDiv.style.display === "none") {
        cellsDiv.style.display = "block";
    } else {
        cellsDiv.style.display = "none";
    }
}


function showCellInfo(sectionName, subsectionName, cellName) {
    var cellInfo = document.getElementById(`cellInfo-${sectionName}-${subsectionName}-${cellName}`);
    if (cellInfo.style.display === "none") {
        cellInfo.style.display = "block";
    } else {
        cellInfo.style.display = "none";
    }
}

function closeOtherSections(exceptSectionId) {
    var sections = document.querySelectorAll(".osnova");
    sections.forEach(function (section) {
        if (section.id !== exceptSectionId) {
            section.style.display = "none";
        }
    });
}





function hideOtherSections(exceptSectionId) {
    // Get all sections except the one specified by exceptSectionId
    var sections = document.querySelectorAll(".infomation");
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


//----------------------------------------------------------------------------------------------------------
//----------------------------------------------------------------------------------------------------------
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

button {
    margin: 13px;
    padding: 15px;
    border-radius: 10%;
    background-color: #d3be25f2;
}

.forplus {
    float: right;
    margin-right: 9%;
}

.button-clicked {
    background-color: #14d424; /* Замените на желаемый цвет */
    color: rgb(0, 0, 0); /* Замените на желаемый цвет текста */
}

.subsection-button-clicked {
    background-color: #4CAF50; /* Замените на желаемый цвет */
    color: white; /* Замените на желаемый цвет текста */
}

.osnova {
    display: flex;
    justify-content: center;
}

.gen_info_section {
    display: flex;
}

.modal {
    display: none;
    position: fixed;
    z-index: 1;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0, 0, 0, 0.7);
}

.modal-content {
    background-color: #fff;
    margin: 10% auto;
    padding: 20px;
    border: 1px solid #888;
    width: 50%;
}

.close {
    color: #aaa;
    float: right;
    font-size: 28px;
    font-weight: bold;
    cursor: pointer;
}

.forosnova {
    display: flex;
    justify-content: space-around;
}

.deck {
    margin-left: 2%;
}

/* внешка для ячеек */


.cellname {
    display: inline-block;
    padding: 1%;
    border-radius: 20px; /* Регулируйте радиус капсулы */
    background: linear-gradient(to bottom, #3498db, #2c3e50); /* Градиент от синего до темно-синего */
    color: #fff; /* Цвет текста (белый в данном случае) */
    width: 92%;
    margin-bottom: 0rem;
}

.celldescrip {
    display: inline-block;
    padding: 1%;
    border-radius: 20px; /* Регулируйте радиус капсулы */
    background: linear-gradient(to right, #3498db, #2c3e50); /* Градиент от синего до темно-синего */
    color: #fff; /* Цвет текста (белый в данном случае) */
    width: 92%;
}

.cellcoment {
    height: 30px;
}
</style>




<main>
    <h1 class="sitename">Survzilla</h1>
    <h1>{$project.first_name}</h1>
    <Link to="/glav" class="btn btn-danger">Выйти из проекта</Link>
    <div class="top">
        <h1>Edit Project: { $project.first_name } { $project.last_name } vessel name: { $project.vessel_name }</h1>
    </div>
    <!--Навигацияа-->
    <div class="deck">
        <button on:click={showIntroductionSections}>INTRODUCTION</button>
        <button on:click={showHullSections}>Hull, DECK & SUPERSTRUCTURE</button>
        <button on:click={showAboveSections}>ABOVE</button>
        <button on:click={showBelowSections}>BELOW</button>
        <button on:click={showCathodicProtectionSections}>CATHODIC PROTECTION</button>
        <button on:click={showHelmStationSections}>HELM STATION & NAVIGATIONAL</button>
        <button on:click={showCabinInteriorSections}>CABIN INTERIOR APPOINTMENTS</button>
        <button on:click={showElectricalSystemsSections}>ELECTRICAL SYSTEMS</button>
        <button on:click={showInboardPropulsionSections}>INBOARD PROPULSION SYSTEM</button>
        <button on:click={showSteeringSystemSections}>STEERING SYSTEM</button>
        <button on:click={showTankageSections}>TANKAGE</button>
        <button on:click={showSafetyEquipmentSections}>SAFETY EQUIPMENT</button>
        <button class="forplus" on:click={showCreateSectionModal}>+</button>
        
        {#each $project.sections as section (section.id)}
            <button id={`${section.name}-button`} on:click={() => showSubsections(section.name)}>{section.name}</button>
            <div id={`subsections-${section.name}`} style="display: none;" class="osnova">
                <h1 class="forosnova">{section.name}</h1>
                <button class="forplus" on:click={() => showCreateSubsectionModal(section.name)}>+</button>
                {#if section.subsections}
                    {#each section.subsections as subsection (subsection.id)}
                        <button on:click={() => showCells(section.name, subsection.name)}>{subsection.name}</button>
                        <div id={`cells-${section.name}-${subsection.name}`} style="display: none;">
                            <h1 class="forosnova">{subsection.name}</h1>
                            <button class="forplus" on:click={() => showCreateCellModal(section.name, subsection.name)}>+</button>
                            {#if subsection.cells}
                                {#each subsection.cells as cell (cell.id)}
                                    <div class="cell">
                                        <button on:click={() => showCellInfo(section.name, subsection.name, cell.name)}>{cell.name}</button>
                                        <div id={`cellInfo-${section.name}-${subsection.name}-${cell.name}`} style="display: none;">
                                            <!-- Вывод информации о ячейке -->
                                            <p class="cellname">{cell.name}</p>
                                            <div class="celldescrip">
                                                <p class=""><strong>Description:</strong> {cell.description}</p>
                                                <ul>{cell.comment}</ul>
                                                <ul>{cell.rating}</ul>
                                                <form action={`http://your-api-url/add_comment`} method="post">
                                                    <label for="cell_comment">Комментарий:</label>
                                                    <textarea class="cellcoment" name="cell_comment" id="cell_comment" rows="4" required></textarea>
                                                    <button type="submit">Добавить комментарий</button>
                                                </form>
                                                <form action={`http://your-api-url/add_rating`} method="post">
                                                    <label for="cell_rating">Рейтинг:</label>
                                                    <select name="cell_rating" id="cell_rating" required>
                                                        <option value="good">Хорошо</option>
                                                        <option value="average">Средне</option>
                                                        <option value="bad">Плохо</option>
                                                    </select>
                                                    <button type="submit">Выбрать рейтинг</button>
                                                </form>
                                            </div>
                                            <h2>Добавить комментарий к ячейке</h2>
                                            <form action={`http://your-api-url/add_comment`} method="post">
                                                <label for="cell_comment">Комментарий:</label>
                                                <textarea class="cellcoment" name="cell_comment" id="cell_comment" rows="4" required></textarea>
                                                <button type="submit">Добавить комментарий</button>
                                            </form>
                                            <h2>Выбрать рейтинг ячейки</h2>
                                            <form action={`http://your-api-url/add_rating`} method="post">
                                                <label for="cell_rating">Рейтинг:</label>
                                                <select name="cell_rating" id="cell_rating" required>
                                                    <option value="good">Хорошо</option>
                                                    <option value="average">Средне</option>
                                                    <option value="bad">Плохо</option>
                                                </select>
                                                <button type="submit">Выбрать рейтинг</button>
                                            </form>
                                            <!-- Добавьте другую информацию о ячейке, если есть -->
                                        </div>
                                    </div>
                                {/each}
                            {/if}
                        </div>
                    {/each}
                {/if}
            </div>
        {/each}
        <div class="osnova" style="display: none;" id="introduction-sections">
            <h1 class="forosnova">Introduction sections</h1>
            <button on:click={() => showSubsection('gen_info')}>Genera Vessel Info</button>
            <button on:click={() => showSubsection('certification')}>CERTIFICATION</button>
            <button on:click={() => showSubsection('purpose_of_survey')}>PURPOSE OF SURVEY</button>
            <button on:click={() => showSubsection('circumstances_of_survey')}>CIRCUMSTANCES OF SURVEY</button>
            <button on:click={() => showSubsection('report_file_no')}>REPORT FILE NO</button>
            <button on:click={() => showSubsection('surveyor_qualifications')}>SURVEYOR QUALIFICATIONS</button>
            <button on:click={() => showSubsection('intended_use')}>INTENDED USE</button>
        </div>
        <div class="osnova" style="display: none;" id="hull-sections">
            <h1 class="forosnova">Hull sections</h1>
            <button on:click={() => showSubsection('layout_overview')}>LAYOUT OVERVIEW</button>
            <button on:click={() => showSubsection('design')}>DESIGN</button>
            <button on:click={() => showSubsection('deck')}>DECK</button>
            <button on:click={() => showSubsection('structural_members')}>STRUCTURAL MEMBERS</button>
            <button on:click={() => showSubsection('bottom_paint')}>BOTTOM PAINT</button>
            <button on:click={() => showSubsection('blister_comment')}>BLISTER COMMENT</button>
            <button on:click={() => showSubsection('transom')}>TRANSOM</button>
        </div>
        <div class="osnova" style="display: none;" id="above-sections">
            <h1 class="forosnova">Above sections</h1>
            <button on:click={() => showSubsection('deck_floor_plan')}>DECK FLOOR PLAN</button>
            <button on:click={() => showSubsection('anchor_platform')}>ANCHOR PLATFORM</button>
            <button on:click={() => showSubsection('toe_rails')}>TOE RAILS</button>
            <button on:click={() => showSubsection('mooring_hardware')}>MOORING HARDWARE</button>
            <button on:click={() => showSubsection('hatches')}>HATCHES</button>
            <button on:click={() => showSubsection('exterior_seating')}>EXTERIOR SEATING</button>
            <button on:click={() => showSubsection('cockpit_equipment')}>COCKPIT EQUIPMENT</button>
            <button on:click={() => showSubsection('ngine_hatch')}>ENGINE HATCH</button>
            <button on:click={() => showSubsection('above_draw_water_line')}>ABOVE DRAW WATER LINE</button>
            <button on:click={() => showSubsection('boarding_ladder')}>BOARDING LADDER</button>
            <button on:click={() => showSubsection('swim_platform')}>SWIM PLATFORM</button>
        </div>
        <div class="osnova" style="display: none;" id="below-sections">
            <h1 class="forosnova">Below sections</h1>
            <button on:click={() => showSubsection('below_draw_water')}>BELOW DRAW WATER</button>
            <button on:click={() => showSubsection('thru_hull_strainers')}>THRU HULL STRAINERS</button>
            <button on:click={() => showSubsection('transducer')}>TRANSDUCER(S)</button>
            <button on:click={() => showSubsection('sea_valves')}>SEA VALVES</button>
            <button on:click={() => showSubsection('sea_strainers')}>SEA STRAINERS</button>
            <button on:click={() => showSubsection('trim_tabs')}>TRIM TABS</button>
            <button on:click={() => showSubsection('note')}>NOTE</button>
        </div>
        <div class="osnova" style="display: none;" id="cathodic-protection-sections">
            <h1 class="forosnova">Cathodic protection sections</h1>
            <button on:click={() => showSubsection('bonding_system')}>BONDING SYSTEM</button>
            <button on:click={() => showSubsection('anodes')}>ANODES</button>
            <button on:click={() => showSubsection('lightning_protection')}>LIGHTNING PROTECTION</button>
            <button on:click={() => showSubsection('additional_remarks')}>ADDITIONAL REMARKS</button>
        </div>
        <div class="osnova" style="display: none;" id="helm-station-sections">
            <h1 class="forosnova">Helm station sections</h1>
            <button on:click={() => showSubsection('helm_station')}>HELM STATION</button>
            <button on:click={() => showSubsection('throttle_shift_controls')}>THROTTLE & SHIFT CONTROLS</button>
            <button on:click={() => showSubsection('engine_room_blowers')}>ENGINE ROOM BLOWERS</button>
            <button on:click={() => showSubsection('engine_status')}>ENGINE STATUS</button>
            <button on:click={() => showSubsection('other_electronics_controls')}>OTHER ELECTRONICS & CONTROLS</button>
        </div>
        <div class="osnova" style="display: none;" id="cabin-interior-sections">
            <h1 class="forosnova">Cabin interior sections</h1>
            <button on:click={() => showSubsection('entertainment_berthing')}>ENTERTAINMENT BERTHING & SALON</button>
            <button on:click={() => showSubsection('interior_lighting')}>INTERIOR LIGHTING</button>
            <button on:click={() => showSubsection('galley_dinette')}>GALLEY/DINETTE & ACCESSORIES</button>
            <button on:click={() => showSubsection('water_closets')}>WATER CLOSET(S)</button>
            <button on:click={() => showSubsection('climate_control')}>CLIMATE CONTROL</button>
        </div>
        
        <div class="osnova" style="display: none;" id="electrical-systems-sections">
            <h1 class="forosnova">Electrical systems sections</h1>
            <button on:click={() => showSubsection('dc_systems_type')}>DIRECT CURRENT SYSTEM(S) TYPE</button>
            <button on:click={() => showSubsection('ac_systems')}>ALTERNATING CURRENT (A.C.) SYSTEM(S)</button>
            <button on:click={() => showSubsection('generator')}>GENERATOR</button>
        </div>
        
        <div class="osnova" style="display: none;" id="inboard-propulsion-sections">
            <h1 class="forosnova">Inboard propulsion sections</h1>
            <button on:click={() => showSubsection('engines')}>ENGINE(S)</button>
            <button on:click={() => showSubsection('serial_numbers')}>SERIAL NUMBERS</button>
            <button on:click={() => showSubsection('engine_hours')}>ENGINE(S) HOURS</button>
            <button on:click={() => showSubsection('other_note')}>OTHER NOTE</button>
            <button on:click={() => showSubsection('reverse_gears')}>REVERSE GEAR(S)</button>
            <button on:click={() => showSubsection('shafting_propellers')}>SHAFTING & PROPELLER(S)</button>
        </div>
        
        <div class="osnova" style="display: none;" id="steering-system-sections">
            <h1 class="forosnova">Steering system sections</h1>
            <button on:click={() => showSubsection('manufacture')}>MANUFACTURE</button>
            <button on:click={() => showSubsection('steering_components')}>STEERING SYSTEM COMPONENTS</button>
        </div>
        
        <div class="osnova" style="display: none;" id="tankage-sections">
            <h1 class="forosnova">Tankage sections</h1>
            <button on:click={() => showSubsection('fuel_tank')}>FUEL TANK(S) & PIPING</button>
            <button on:click={() => showSubsection('potable_water_system')}>POTABLE WATER SYSTEM & WATER HEATER</button>
            <button on:click={() => showSubsection('holding_tank_black_water')}>HOLDING TANK(S)-BLACK WATER</button>
        </div>
        
        <div class="osnova" style="display: none;" id="safety-equipment-sections">
            <h1 class="forosnova">Safety equipment sections</h1>
            <button on:click={() => showSubsection('navigational_lights')}>NAVIGATIONAL LIGHTS</button>
            <button on:click={() => showSubsection('life_jackets')}>LIFE JACKETS (P.F.D,'S)</button>
            <button on:click={() => showSubsection('throwable_pfd')}>THROWABLE TYPE P.F.D</button>
            <button on:click={() => showSubsection('visual_distress_signals')}>VISUAL DISTRESS SIGNALS</button>
            <button on:click={() => showSubsection('sound_devices')}>SOUND DEVICES</button>
            <button on:click={() => showSubsection('uscg_placards')}>U.S.C.G. PLACARDS</button>
            <button on:click={() => showSubsection('flame_arrestors')}>FLAME ARRESTOR(S)</button>
            <button on:click={() => showSubsection('engine_ventilation')}>ENGINE VENTILATION</button>
            <button on:click={() => showSubsection('ignition_protection')}>IGNITION PROTECTION</button>
            <button on:click={() => showSubsection('inland_navigational_rule_book')}>INLAND NAVIGATIONAL RULE BOOK</button>
            <button on:click={() => showSubsection('waste_management_plan')}>WASTE MANAGEMENT PLAN</button>
            <button on:click={() => showSubsection('fire_fighting_equipment')}>FIRE FIGHTING EQUIPMENT</button>
            <button on:click={() => showSubsection('bilge_pumps')}>BILGE PUMPS</button>
            <button on:click={() => showSubsection('ground_tackle_windlass')}>GROUND TACKLE & WINDLASS</button>
            <button on:click={() => showSubsection('auxiliary_safety_equipment')}>AUXILIARY SAFETY EQUIPMENT</button>
        </div>


    <!--Subsection section safety-equipment-->
    <!--navigational_lights -->
    <div id="navigational_lights-section" style="display: none;" class="infomation">
        <h2>navigational_lights:</h2>
        <ul>
            {#each navigational_lights_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('navigational_lights', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="navigational_lights" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- life_jackets -->
    <div id="life_jackets-section" style="display: none;" class="infomation">
        <h2>life jackets:</h2>
        <ul>
            {#each project.life_jackets_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('life_jackets', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="life_jackets" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- throwable_pfd -->
    <div id="throwable_pfd-section" style="display: none;" class="infomation">
        <h2>THROWABLE TYPE P.F.D:</h2>
        <ul>
            {#each project.throwable_pfd_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('throwable_pfd', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="throwable_pfd" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- visual_distress_signals -->
    <div id="visual_distress_signals-section" style="display: none;" class="infomation">
        <h2>visual distress signals:</h2>
        <ul>
            {#each project.visual_distress_signals_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('visual_distress_signals', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="visual_distress_signals" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- sound_devices -->
    <div id="sound_devices-section" style="display: none;" class="infomation">
        <h2>sound devices:</h2>
        <ul>
            {#each project.sound_devices_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('sound_devices', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="sound_devices" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- uscg_placards -->
    <div id="uscg_placards-section" style="display: none;" class="infomation">
        <h2>uscg placards:</h2>
        <ul>
            {#each project.uscg_placards_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('uscg_placards', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="uscg_placards" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- flame_arrestors -->
    <div id="flame_arrestors-section" style="display: none;" class="infomation">
        <h2>flame arrestors:</h2>
        <ul>
            {#each project.flame_arrestors_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('flame_arrestors', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="flame_arrestors" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- engine_ventilation -->
    <div id="engine_ventilation-section" style="display: none;" class="infomation">
        <h2>engine ventilation:</h2>
        <ul>
            {#each project.engine_ventilation_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('engine_ventilation', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="engine_ventilation" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- ignition_protection -->
    <div id="ignition_protection-section" style="display: none;" class="infomation">
        <h2>ignition protection:</h2>
        <ul>
            {#each project.ignition_protection_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('ignition_protection', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="ignition_protection" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- inland_navigational_rule_book -->
    <div id="inland_navigational_rule_book-section" style="display: none;" class="infomation">
        <h2>inland navigational rule book:</h2>
        <ul>
            {#each project.inland_navigational_rule_book_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('inland_navigational_rule_book', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="inland_navigational_rule_book" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- waste_management_plan -->
    <div id="waste_management_plan-section" style="display: none;" class="infomation">
        <h2>waste management plan:</h2>
        <ul>
            {#each project.waste_management_plan_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('waste_management_plan', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="waste_management_plan" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- fire_fighting_equipment -->
    <div id="fire_fighting_equipment-section" style="display: none;" class="infomation">
        <h2>fire fighting equipment:</h2>
        <ul>
            {#each project.fire_fighting_equipment_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('fire_fighting_equipment', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="fire_fighting_equipment" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- bilge_pumps -->
    <div id="bilge_pumps-section" style="display: none;" class="infomation">
        <h2>bilge pumps:</h2>
        <ul>
            {#each project.bilge_pumps_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('bilge_pumps', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="bilge_pumps" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- ground_tackle_windlass -->
    <div id="ground_tackle_windlass-section" style="display: none;" class="infomation">
        <h2>ground tackle windlass:</h2>
        <ul>
            {#each project.ground_tackle_windlass_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('ground_tackle_windlass', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="ground_tackle_windlass" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- auxiliary_safety_equipment -->
    <div id="auxiliary_safety_equipment-section" style="display: none;" class="infomation">
        <h2>auxiliary safety equipment:</h2>
        <ul>
            {#each project.auxiliary_safety_equipment_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('auxiliary_safety_equipment', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="auxiliary_safety_equipment" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!--Finish subsection section tankage-->


    <!--Subsection section tankage-->
    <!-- holding_tank_black_water -->
    <div id="holding_tank_black_water-section" style="display: none;" class="infomation">
        <h2>holding tank black water:</h2>
        <ul>
            {#each project.holding_tank_black_water_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('holding_tank_black_water', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="holding_tank_black_water" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- potable_water_system -->
    <div id="potable_water_system-section" style="display: none;" class="infomation">
        <h2>potable water system:</h2>
        <ul>
            {#each project.potable_water_system_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('potable_water_system', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="potable_water_system" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- fuel_tank -->
    <div id="fuel_tank-section" style="display: none;" class="infomation">
        <h2>fuel tank:</h2>
        <ul>
            {#each project.fuel_tank_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('fuel_tank', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="fuel_tank" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!--Finish subsection section steering-system-->


    <!--Subsection section steering-system-->
    <!-- steering_components -->
    <div id="steering_components-section" style="display: none;" class="infomation">
        <h2>steering components:</h2>
        <ul>
            {#each project.steering_components_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('steering_components', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="steering_components" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- manufacture -->
    <div id="manufacture-section" style="display: none;" class="infomation">
        <h2>manufacture:</h2>
        <ul>
            {#each project.manufacture_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('manufacture', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="manufacture" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!--Finish subsection section steering-system-->


    <!--Subsection section inboard-propulsion-->
    <!-- shafting_propellers -->
    <div id="shafting_propellers-section" style="display: none;" class="infomation">
        <h2>shafting propellers:</h2>
        <ul>
            {#each project.shafting_propellers_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('shafting_propellers', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="shafting_propellers" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- reverse_gears -->
    <div id="reverse_gears-section" style="display: none;" class="infomation">
        <h2>reverse gears:</h2>
        <ul>
            {#each project.reverse_gears_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('reverse_gears', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="reverse_gears" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- other_note -->
    <div id="other_note-section" style="display: none;" class="infomation">
        <h2>other note:</h2>
        <ul>
            {#each project.other_note_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('other_note', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="other_note" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- engine_hours-->
    <div id="engine_hours-section" style="display: none;" class="infomation">
        <h2>engine hours:</h2>
        <ul>
            {#each project.engine_hours_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('engine_hours', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="engine_hours" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- serial_numbers -->
    <div id="serial_numbers-section" style="display: none;" class="infomation">
        <h2>serial numbers:</h2>
        <ul>
            {#each project.serial_numbers_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('serial_numbers', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="serial_numbers" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- engines -->
    <div id="engines-section" style="display: none;" class="infomation">
        <h2>engines:</h2>
        <ul>
            {#each project.engines_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('engines', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="engines" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!--Finish subsection section inboard-propulsion-->

    <!--Subsection section electrical-systems-->
    <!-- ac_systems -->
    <div id="generator-section" style="display: none;" class="infomation">
        <h2>generator:</h2>
        <ul>
            {#each project.generator_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('generator', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="generator" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- ac_systems -->
    <div id="ac_systems-section" style="display: none;" class="infomation">
        <h2>ac systems:</h2>
        <ul>
            {#each project.ac_systems_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('ac_systems', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="ac_systems" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- dc_systems_type -->
    <div id="dc_systems_type-section" style="display: none;" class="infomation">
        <h2>dc systems type:</h2>
        <ul>
            {#each project.dc_systems_type_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('dc_systems_type', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="dc_systems_type" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!--Finish subsection section electrical-systems-->

    <!--Subsection section cabin-interior-->
    <!-- climate_control -->
    <div id="climate_control-section" style="display: none;" class="infomation">
        <h2>climate control:</h2>
        <ul>
            {#each project.climate_control_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('climate_control', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="climate_control" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- water_closets -->
    <div id="water_closets-section" style="display: none;" class="infomation">
        <h2>water closets:</h2>
        <ul>
            {#each project.water_closets_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('water_closets', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="water_closets" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- galley_dinette -->
    <div id="galley_dinette-section" style="display: none;" class="infomation">
        <h2>galleyn dinette:</h2>
        <ul>
            {#each project.galley_dinette_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('galley_dinette', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="galley_dinette" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- interior_lighting -->
    <div id="interior_lighting-section" style="display: none;" class="infomation">
        <h2>interior lighting:</h2>
        <ul>
            {#each project.interior_lighting_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('interior_lighting', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="interior_lighting" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- entertainment_berthing -->
    <div id="entertainment_berthing-section" style="display: none;" class="infomation">
        <h2>entertainment berthing:</h2>
        <ul>
            {#each project.entertainment_berthing_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('entertainment_berthing', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="entertainment_berthing" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>

    <!--Finish subsection section cabin-interior-->

    <!--Subsection section helm-station-->
    <!-- other_electronics_controls -->
    <div id="other_electronics_controls-section" style="display: none;" class="infomation">
        <h2>other electronics controls:</h2>
        <ul>
            {#each project.other_electronics_controls_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('other_electronics_controls', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="other_electronics_controls" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- engine_status -->
    <div id="engine_status-section" style="display: none;" class="infomation">
        <h2>engine status:</h2>
        <ul>
            {#each project.engine_status_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('engine_status', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="engine_status" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- engine_room_blowers -->
    <div id="engine_room_blowers-section" style="display: none;" class="infomation">
        <h2>engine room blowers:</h2>
        <ul>
            {#each project.engine_room_blowers_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('engine_room_blowers', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="engine_room_blowers" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- throttle_shift_controls -->
    <div id="throttle_shift_controls-section" style="display: none;" class="infomation">
        <h2>throttle shift controls:</h2>
        <ul>
            {#each project.throttle_shift_controls_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('throttle_shift_controls', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="throttle_shift_controls" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- helm_station -->
    <div id="helm_station-section" style="display: none;" class="infomation">
        <h2>helm station:</h2>
        <ul>
            {#each project.helm_station_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('helm_station', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="helm_station" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>

    <!--Finish subsection section helm-station-->


    <!--Subsection section cathodic-protection-->
    <!-- additional_remarks -->
    <div id="additional_remarks-section" style="display: none;" class="infomation">
        <h2>additional remarks:</h2>
        <ul>
            {#each project.additional_remarks_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('additional_remarks', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="additional_remarks" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- lightning_protection -->
    <div id="lightning_protection-section" style="display: none;" class="infomation">
        <h2>lightning protection:</h2>
        <ul>
            {#each project.lightning_protection_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('lightning_protection', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="lightning_protection" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- anodes -->
    <div id="anodes-section" style="display: none;" class="infomation">
        <h2>anodes:</h2>
        <ul>
            {#each project.anodes_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('anodes', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="anodes" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- bonding_system -->
    <div id="bonding_system-section" style="display: none;" class="infomation">
        <h2>bonding system:</h2>
        <ul>
            {#each project.bonding_system_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('bonding_system', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="bonding_system" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!--Finish subsection section below-->


    <!--Subsection section below-->
    <!-- note -->
    <div id="note-section" style="display: none;" class="infomation">
        <h2>note:</h2>
        <ul>
            {#each project.note_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('note', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="note" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- trim_tabs -->
    <div id="trim_tabs-section" style="display: none;" class="infomation">
        <h2>trim tabs:</h2>
        <ul>
            {#each project.trim_tabs_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('trim_tabs', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="trim_tabs" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- sea_strainers -->
    <div id="sea_strainers-section" style="display: none;" class="infomation">
        <h2>sea strainers:</h2>
        <ul>
            {#each project.sea_strainers_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('sea_strainers', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="sea_strainers" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- sea_valves -->
    <div id="sea_valves-section" style="display: none;" class="infomation">
        <h2>sea valves:</h2>
        <ul>
            {#each project.sea_valves_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('sea_valves', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="sea_valves" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- transducer -->
    <div id="transducer-section" style="display: none;" class="infomation">
        <h2>transducer:</h2>
        <ul>
            {#each project.transducer_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('transducer', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="transducer" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- thru_hull_strainers -->
    <div id="thru_hull_strainers-section" style="display: none;" class="infomation">
        <h2>thru hull strainers:</h2>
        <ul>
            {#each project.thru_hull_strainers_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('thru_hull_strainers', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="thru_hull_strainers" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- below_draw_water -->
    <div id="below_draw_water-section" style="display: none;" class="infomation">
        <h2>below draw water:</h2>
        <ul>
            {#each project.below_draw_water_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('below_draw_water', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="below_draw_water" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>

    <!--Finish subsection section below-->



    <!--Subsection section above-->
    <!-- swim_platform -->
    <div id="swim_platform-section" style="display: none;" class="infomation">
        <h2>swim platform:</h2>
        <ul>
            {#each project.swim_platform_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('swim_platform', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="swim_platform" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- boarding_ladder -->
    <div id="boarding_ladder-section" style="display: none;" class="infomation">
        <h2>boarding ladder:</h2>
        <ul>
            {#each project.boarding_ladder_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('boarding_ladder', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="boarding_ladder" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- above_draw_water_line -->
    <div id="above_draw_water_line-section" style="display: none;" class="infomation">
        <h2>above draw water line:</h2>
        <ul>
            {#each project.above_draw_water_line_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('above_draw_water_line', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="above_draw_water_line" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- engine_hatch -->
    <div id="engine_hatch-section" style="display: none;" class="infomation">
        <h2>engine hatch:</h2>
        <ul>
            {#each project.engine_hatch_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('engine_hatch', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="engine_hatch" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- cockpit_equipment -->
    <div id="cockpit_equipment-section" style="display: none;" class="infomation">
        <h2>ecockpit equipment:</h2>
        <ul>
            {#each project.cockpit_equipment_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('cockpit_equipment', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="cockpit_equipment" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- exterior_seating -->
    <div id="exterior_seating-section" style="display: none;" class="infomation">
        <h2>exterior seating:</h2>
        <ul>
            {#each project.exterior_seating_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('exterior_seating', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="exterior_seating" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- hatches -->
    <div id="hatches-section" style="display: none;" class="infomation">
        <h2>hatches:</h2>
        <ul>
            {#each project.hatches_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('hatches', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="hatches" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- mooring_hardware -->
    <div id="mooring_hardware-section" style="display: none;" class="infomation">
        <h2>mooring hardware:</h2>
        <ul>
            {#each project.mooring_hardware_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('mooring_hardware', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="mooring_hardware" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- toe_rails -->
    <div id="toe_rails-section" style="display: none;" class="infomation">
        <h2>toe rails:</h2>
        <ul>
            {#each project.toe_rails_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('toe_rails', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="toe_rails" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- anchor_platform -->
    <div id="anchor_platform-section" style="display: none;" class="infomation">
        <h2>anchor platform:</h2>
        <ul>
            {#each project.anchor_platform_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('anchor_platform', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="anchor_platform" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- deck_floor_plan -->
    <div id="deck_floor_plan-section" style="display: none;" class="infomation">
        <h2>deck floor plan:</h2>
        <ul>
            {#each project.deck_floor_plan_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('deck_floor_plan', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="deck_floor_plan" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!--Finish subsection section above-->


    <!--Subsection section hull-->
    <!-- transom -->
    <div id="transom-section" style="display: none;" class="infomation">
        <h2>transom:</h2>
        <ul>
            {#each project.transom_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('transom', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="transom" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- blister_comment -->
    <div id="blister_comment-section" style="display: none;" class="infomation">
        <h2>blister comment:</h2>
        <ul>
            {#each project.blister_comment_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('blister_comment', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="blister_comment" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- bottom_paint -->
    <div id="bottom_paint-section" style="display: none;" class="infomation">
        <h2>bottom paint:</h2>
        <ul>
            {#each project.bottom_paint_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('bottom_paint', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="bottom_paint" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- structural_members -->
    <div id="structural_members-section" style="display: none;" class="infomation">
        <h2>structural members:</h2>
        <ul>
            {#each project.structural_members_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('structural_members', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="structural_members" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- deck -->
    <div id="deck-section" style="display: none;" class="infomation">
        <h2>deck:</h2>
        <ul>
            {#each project.deck_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('deck', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="deck" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- design -->
    <div id="design-section" style="display: none;" class="infomation">
        <h2>design:</h2>
        <ul>
            {#each project.design_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('design', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="design" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- layout_overview -->
    <div id="layout_overview-section" style="display: none;" class="infomation">
        <h2>layout overview:</h2>
        <ul>
            {#each project.layout_overview_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('layout_overview', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="layout_overview" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!--Finishsubsection section hull-->


    <!--Subsection section INTRODUCTION-->
    <!-- intended_use -->
    <div id="intended_use-section" style="display: none;" class="infomation">
        <h2>intended use:</h2>
        <ul>
            {#each project.intended_use_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('intended_use', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="intended_use" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- surveyor_qualifications -->
    <div id="surveyor_qualifications-section" style="display: none;" class="infomation">
        <h2>surveyor qualifications:</h2>
        <ul>
            {#each project.surveyor_qualifications_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('surveyor_qualifications', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="surveyor_qualifications" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- report_file_no -->
    <div id="report_file_no-section" style="display: none;" class="infomation">
        <h2>report file no:</h2>
        <ul>
            {#each project.report_file_no_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('report_file_no', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="report_file_no" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!-- circumstances_of_survey -->
    <div id="circumstances_of_survey-section" style="display: none;" class="infomation">
        <h2>Circumstances of Survey:</h2>
        <ul>
            {#each project.circumstances_of_survey_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('circumstances_of_survey', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="circumstances_of_survey" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- purpose_of_survey -->
    <div id="purpose_of_survey-section" style="display: none;" class="infomation">
        <h2>purpose of survey:</h2>
        <ul>
            {#each project.purpose_of_survey_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('purpose_of_survey', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="purpose_of_survey" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    
    <!-- certification -->
    <div id="certification-section" style="display: none;" class="infomation">
        <h2>certification:</h2>
        <ul>
            {#each project.certification_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('certification', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="certification" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!--Finish subsection section INTRODUCTION-->


    <!--Subsection section INF-->
    <!-- Секция Gen Info -->
    <div id="gen_info-section" class="infomation" style="display: none;">
        <h2>Gen Info:</h2>
        <div>
            <p><strong>City:</strong> { project.city }</p>
            <p><strong>Phone:</strong> { project.phone }</p>
            <p><strong>Post:</strong> { project.post }</p>
            <p><strong>vessel name:</strong> { project.vessel_name }</p>
        </div>
        <ul>
            {#each project.gen_info_steps as step}
                <li>{step}</li>
                <form on:submit|preventDefault={() => deleteStep('gen_info', step)}>
                    <input type="hidden" name="step_to_delete" value={step} />
                    <input type="hidden" name="section" value="gen_info" />
                    <button type="submit">Delete</button>
                </form>
            {/each}
        </ul>
    </div>
    <!--Finish subsection section INF-->
    
    <!---------------------            Форма для добавления шага в текущий раздел                --------------------->
    <div id="add-step-form-container" style="display: none;">
        <h2>Add Step:</h2>
        <form on:submit|preventDefault={() => addStep('gen_info')}><!--<form method="POST" action="/edit_project/{{ project_id }}/add_step">-->
            <label for="step_description">Step Description:</label>
            <input type="text" id="step_description" name="step_description">

            <label for="image">Image:</label>
            <input type="file" id="image" name="image">
            
            <input type="hidden" name="section" id="current_section" value="">
            <input type="submit" value="Add Step">
        </form>
    </div>

    <!--  тестирую      -->
    <!-- <div id="createSectionModal" class="modal">
    <!--     <div class="modal-content">
    <!--         <span class="close" onclick="closeCreateSectionModal()">&times;</span>
    <!--         <h2 class="formablack">Create New Section</h2>
    <!--         <form action="{url_for('add_section', { project_id })}" method="post" on:submit|preventDefault={handleSubmitSection}><!-- <form action="{{ url_for('add_section', project_id=project_id) }}" method="post">-->
    <!--             <label class="formablack" for="section_name">Имя раздела:</label>
    <!--             <input type="text" name="section_name" id="section_name" required>
    <!--             <button type="submit">Name section</button>
    <!--         </form>
    <!--     </div>
    <!-- </div>
    <!-- {% for section in project.sections %}
    <!--     <div id="createSubsectionModal" class="modal">
    <!--     <div class="modal-content">
    <!--         <span class="close" onclick="closeCreateSubsectionModal()">&times;</span>
    <!--         <h2 class="formablack">Create New Subsection</h2>
    <!--         <!-- <form action="{{ url_for('add_subsection', project_id=project_id) }}" method="post">-->
    <!--             <input type="hidden" name="section_name" id="section_name_input" value="">
    <!--             <label class="formablack" for="subsection_name">Имя подраздела:</label>
    <!--             <input type="text" name="subsection_name" id="subsection_name" required>
    <!--             <button class="formablack" type="submit">Добавить подраздел</button>
    <!--         </form>
    <!--     </div>
    <!--     </div>
    <!--     {% for subsection in section.subsections %}
    <!--         <div id="createCellModal" class="modal">
    <!--             <div class="modal-content">
    <!--                 <span class="close" onclick="closeCreateCellModal()">&times;</span>
    <!--                 <h2 class="formablack">Create New Cell</h2>
    <!--                 <!-- <form action="{{ url_for('add_cell', project_id=project_id) }}" method="post">-->
    <!--                     <input type="hidden" name="section_name" id="section_name_input" value="">
    <!--                     <input type="hidden" name="subsection_name" id="subsection_name_input" value="">
    <!--                     <label class="formablack" for="cell_name">Имя ячейки:</label>
    <!--                     <input type="text" name="cell_name" id="cell_name" required>
    <!--                     <label class="formablack" for="cell_description">Описание ячейки:</label>
    <!--                     <textarea class="cellcoment" name="cell_description" id="cell_description" rows="4" required></textarea>
    <!--                     <button class="formablack" type="submit">Добавить ячейку</button>
    <!--                 </form>
    <!--             </div>
    <!--         </div>
    <!--     {% endfor %}
    <!-- {% endfor %}-->
</main>
