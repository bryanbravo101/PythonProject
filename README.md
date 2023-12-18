import streamlit as st
import matplotlib.pyplot as plt
import pandas as pd
import pydeck as pyd

# I thought this was pretty cool, I was really proud of myself for thinking of something like this!
password = "CS230"
passwordEntry = st.text_input("Please enter the password to access confidential information:")
if passwordEntry != password:
    st.warning("(Hint: CS230)")
else:
    # data import
    df = pd.read_csv('bostoncrime2023_7000_sample.csv',
                     header=0,
                     names=['INCIDENT_NUMBER', 'OFFENSE_CODE', 'OFFENSE_CODE_GROUP', 'OFFENSE_DESCRIPTION', 'DISTRICT', 'REPORTING_AREA', 'SHOOTING', 'OCCURRED_ON_DATE', 'YEAR', 'MONTH', 'DAY_OF_WEEK', 'HOUR', 'UCR_PART', 'STREET', 'Lat', 'Long', 'Location'],
                     )

    # heading section
    st.title("Boston Crime")
    st.subheader("By: Bryan Bravo")
    st.image("BostonPolice.jpeg")

    # image from https://www.masslive.com/news/2023/08/boston-police-officer-suspended-for-taking-money-from-wallet-department-says.html
    st.markdown("According to cbsnews.com, 'Boston is a relatively safe city. It ranked as the tenth safest city in a MoneyGeek analysis of FBI crime data. When looking at violent crime, Boston is actually safer than Dallas'.")
    # quote from https://www.cbsnews.com/boston/news/boston-americas-safest-cities-gallup-survey-poll-dallas/
    st.markdown("Let's take a look at the Boston Crime data from January to November 2023.")

    # code to remove an error message on streamlit
    st.set_option('deprecation.showPyplotGlobalUse', False)

    # bar chart
    crimeMonth = df['MONTH'].value_counts().sort_index()
    months = ['Jan', 'Feb', 'Mar', "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov"]
    fig, ax = plt.subplots()
    ax.bar(months, crimeMonth.values, color='blue')
    ax.set_xlabel("Month")
    ax.set_ylabel("Number of Crimes")
    ax.set_title('Reported Crimes from January to November 2023')
    ax.set_yticks(range(0, 1000, 75))

    # display
    st.pyplot(fig)

    # scatter plot
    st.markdown("Now let's see how many crimes happened for each hour of the day:")
    crimeHours = df['HOUR'].value_counts().sort_index()
    fig1, ax = plt.subplots()
    ax.plot(crimeHours.index, crimeHours.values, 'o', markersize=2)
    ax.set_xlabel('Hour of the Day (Military Time)')
    ax.set_xticks(range(0, 24, 2))
    ax.set_ylabel('Number of Crimes')
    ax.set_title(f'Crime Incidents by the Hour')

    # display
    st.pyplot(fig1)

    # dataframe for streamlit
    crimeCounts = df['OFFENSE_DESCRIPTION'].value_counts()
    uniqueCrimes = crimeCounts.index
    crimeCountsValues = crimeCounts.values
    crimeData = {'Offense Description': uniqueCrimes, 'Total Count': crimeCountsValues}
    crimeDF = pd.DataFrame(crimeData)

    # display
    st.markdown("Let's also find out what the most and least common crime in Boston is:")
    st.write(crimeDF)

    # dropdown for selecting districts
    districts = st.multiselect("Which district(s) would you like display on a map?",
                               ["A1", "A15", "A7", "B2", "B3",
                                "C6", "C11", "D4", "D14",
                                "E5", "E13", "E18"])

    # determine if districts are selected, if not, map will not show, message displays.
    if not districts:
        st.warning("Please select at least one district.")
    else:

        # remove missing latitude and longitude values from csv file for map to increase accuracy
        dfMap = df.dropna(subset=['Lat', 'Long'])

        # filter dfMap based on selected districts
        dfMapFiltered = dfMap[dfMap['DISTRICT'].isin(districts)]

        # used https://police.boston.gov/districts/ to match up districts
        if "A1" in districts:
            st.write("You selected A1, Downtown")
        if "A15" in districts:
            st.write("You selected A15, Charlestown")
        if "A7" in districts:
            st.write("You selected A7, East Boston")
        if "B2" in districts:
            st.write("You selected B2, Roxbury")
        if "B3" in districts:
            st.write("You selected B3, Mattapan")
        if "C6" in districts:
            st.write("You selected C6, South Boston")
        if "C11" in districts:
            st.write("You selected C11, Dorchester")
        if "D4" in districts:
            st.write("You selected D4, South End")
        if "D14" in districts:
            st.write("You selected D14, Brighton")
        if "E5" in districts:
            st.write("You selected E5, West Roxbury")
        if "E13" in districts:
            st.write("You selected E13, Jamaica Plain")
        if "E18" in districts:
            st.write("You selected E18, Hyde Park")

        # offense descriptions and their assigned colors, chatGPT was used since 100 RGB values needed to be assigned. Only about 10-15 colors are present, each crime does not have a unique color.
        offenseColors = {
            'M/V - LEAVING SCENE - PROPERTY DAMAGE': [255, 0, 0],
            'ASSAULT - SIMPLE': [0, 255, 0],
            'M/V ACCIDENT - PROPERTY DAMAGE': [0, 0, 255],
            'FRAUD - FALSE PRETENSE / SCHEME': [255, 255, 0],
            'LARCENY THEFT FROM MV - NON-ACCESSORY': [255, 0, 255],
            'MISSING PERSON - NOT REPORTED - LOCATED': [0, 255, 255],
            'LARCENY SHOPLIFTING': [128, 0, 0],
            'LARCENY THEFT OF MV PARTS & ACCESSORIES': [0, 128, 0],
            'PROPERTY - FOUND': [0, 0, 128],
            'INVESTIGATE PROPERTY': [128, 128, 128],
            'INVESTIGATE PERSON': [128, 0, 128],
            'ASSAULT - AGGRAVATED': [255, 255, 255],
            'SICK ASSIST': [192, 192, 192],
            'DEATH INVESTIGATION': [255, 165, 0],
            'VAL - OPERATING AFTER REV/SUSP.': [210, 180, 140],
            'TOWED MOTOR VEHICLE': [240, 128, 128],
            'MISSING PERSON - LOCATED': [128, 0, 128],
            'SICK ASSIST - DRUG RELATED ILLNESS': [255, 69, 0],
            'LARCENY THEFT FROM BUILDING': [0, 255, 255],
            'LIQUOR/ALCOHOL - DRINKING IN PUBLIC': [255, 0, 127],
            'LARCENY THEFT OF BICYCLE': [255, 69, 0],
            'VANDALISM': [128, 0, 0],
            'AUTO THEFT': [0, 255, 255],
            'VERBAL DISPUTE': [255, 69, 0],
            'THREATS TO DO BODILY HARM': [128, 128, 128],
            'FRAUD - CREDIT CARD / ATM FRAUD': [255, 255, 0],
            'PROPERTY - LOST/ MISSING': [0, 128, 0],
            'OPERATING UNDER THE INFLUENCE (OUI) ALCOHOL': [255, 0, 127],
            'FRAUD - WIRE': [255, 165, 0],
            'ROBBERY': [0, 0, 128],
            'M/V ACCIDENT - OTHER': [255, 0, 0],
            'M/V ACCIDENT - OTHER CITY VEHICLE': [255, 0, 0],
            'LARCENY ALL OTHERS': [255, 255, 0],
            'SICK/INJURED/MEDICAL - PERSON': [255, 69, 0],
            'RECOVERED - MV RECOVERED IN BOSTON (STOLEN OUTSIDE BOSTON)': [0, 255, 255],
            'VAL - VIOLATION OF AUTO LAW': [210, 180, 140],
            'SICK/INJURED/MEDICAL - POLICE': [255, 69, 0],
            'TRESPASSING': [255, 0, 127],
            'WEAPON VIOLATION - CARRY/ POSSESSING/ SALE/ TRAFFICKING/ OTHER': [128, 0, 0],
            'WARRANT ARREST - OUTSIDE OF BOSTON WARRANT': [128, 128, 128],
            'PROPERTY - ACCIDENTAL DAMAGE': [0, 255, 0],
            'DRUGS - POSSESSION/ SALE/ MANUFACTURING/ USE': [0, 128, 0],
            'M/V ACCIDENT - PERSONAL INJURY': [255, 0, 0],
            'BALLISTICS EVIDENCE/FOUND': [0, 128, 128],
            'SUDDEN DEATH': [128, 0, 0],
            'HARASSMENT/ CRIMINAL HARASSMENT': [255, 69, 0],
            'INTIMIDATING WITNESS': [255, 69, 0],
            'BURGLARY - RESIDENTIAL': [128, 0, 0],
            'LANDLORD - TENANT': [0, 128, 0],
            'LARCENY PICK-POCKET': [255, 255, 0],
            'SEARCH WARRANT': [0, 128, 128],
            'LICENSE PREMISE VIOLATION': [255, 255, 0],
            'MISSING PERSON': [128, 0, 128],
            'FORGERY / COUNTERFEITING': [255, 255, 0],
            'HARBOR INCIDENT / VIOLATION': [0, 0, 255],
            'FIREARM/WEAPON - FOUND OR CONFISCATED': [0, 128, 0],
            'M/V ACCIDENT - INVOLVING PEDESTRIAN - INJURY': [255, 0, 0],
            'ANIMAL INCIDENTS (DOG BITES, LOST DOG, ETC)': [128, 0, 0],
            'VIOLATION - CITY ORDINANCE': [0, 0, 128],
            'M/V - LEAVING SCENE - PERSONAL INJURY': [255, 0, 0],
            'STOLEN PROPERTY - BUYING / RECEIVING / POSSESSING': [0, 255, 0],
            'M/V PLATES - LOST': [255, 0, 0],
            'FRAUD - WELFARE': [255, 255, 0],
            'FIRE REPORT': [255, 0, 0],
            'BREAKING AND ENTERING (B&E) MOTOR VEHICLE': [255, 0, 0],
            'FUGITIVE FROM JUSTICE': [128, 0, 0],
            'M/V ACCIDENT - INVOLVING BICYCLE - INJURY': [255, 0, 0],
            'AUTO THEFT - MOTORCYCLE / SCOOTER': [0, 255, 0],
            'DISTURBING THE PEACE/ DISORDERLY CONDUCT/ GATHERING CAUSING ANNOYANCE/ NOISY PAR': [255, 69, 0],
            'EVADING FARE': [255, 69, 0],
            'LARCENY PURSE SNATCH - NO FORCE': [255, 255, 0],
            'M/V ACCIDENT - INVOLVING PEDESTRIAN - NO INJURY': [255, 0, 0],
            'M/V ACCIDENT - INVOLVING BICYCLE - NO INJURY': [255, 0, 0],
            'FRAUD - IMPERSONATION': [255, 255, 0],
            'SERVICE TO OTHER AGENCY': [0, 255, 255],
            'M/V ACCIDENT - POLICE VEHICLE': [255, 0, 0],
            'AUTO THEFT - LEASED/RENTED VEHICLE': [0, 255, 0],
            'FIRE REPORT/ALARM - FALSE': [255, 0, 0],
            'NOISY PARTY/RADIO-NO ARREST': [255, 69, 0],
            'WARRANT ARREST - BOSTON WARRANT (MUST BE SUPPLEMENTAL)': [128, 128, 128],
            'GRAFFITI': [0, 0, 255],
            'BURGLARY - COMMERICAL': [128, 0, 0],
            'PROPERTY - LOST THEN LOCATED': [0, 128, 0],
            'DANGEROUS OR HAZARDOUS CONDITION': [255, 0, 0],
            'AFFRAY': [255, 0, 0],
            'VAL - OPERATING W/O AUTHORIZATION LAWFUL': [210, 180, 140],
            'INJURY BICYCLE NO M/V INVOLVED': [255, 0, 0],
            'EXTORTION OR BLACKMAIL': [255, 0, 0],
            'MURDER, NON-NEGLIGENT MANSLAUGHTER': [128, 0, 0],
            'PROPERTY - STOLEN THEN RECOVERED': [0, 255, 0],
            'SUICIDE / SUICIDE ATTEMPT': [128, 0, 0],
            'EMBEZZLEMENT': [255, 255, 0],
            'DRUGS - POSSESSION OF DRUG PARAPHANALIA': [0, 128, 0],
            'ARSON': [255, 0, 128],
            'TRUANCY / RUNAWAY': [255, 128, 128],
            'OBSCENE PHONE CALLS': [0, 255, 128],
            'OTHER OFFENSE': [0, 128, 255],
            'BOMB THREAT': [128, 255, 128],
            'RECOVERED - MV RECOVERED IN BOSTON (STOLEN IN BOSTON) MUST BE SUPPLEMENTAL': [128, 0, 255],
            'PRISONER - SUICIDE / SUICIDE ATTEMPT': [255, 128, 255],
        }

        # display color on map based on offense description
        dfMapFiltered['color'] = dfMapFiltered['OFFENSE_DESCRIPTION'].map(offenseColors.get)

        # map building
        layer = pyd.Layer(
            "ScatterplotLayer",
            data=dfMapFiltered,
            get_position='[Long, Lat]',
            get_radius=25,
            get_fill_color='color',
            pickable=True,
        )

        # allow user to zoom in and out, limited to avoid extra zooming in and unclear map, also recenters map.
        zoomInOut = st.slider('Use this slider to adjust the zoom and recenter the map!', 11, 15, 11)

        # gets mean of lat and long to default map to show Boston
        viewBoston = pyd.ViewState(
            latitude=dfMapFiltered['Lat'].mean(),
            longitude=dfMapFiltered['Long'].mean(),
            zoom=zoomInOut,
            pitch=0,
        )
        # announce to user to hover mouse over dot
        st.markdown("Hover your mouse over a dot to see what the offense description is!")

        # allows user to see offense description based on dot hovered over.
        dotOffense = {"text": "{OFFENSE_DESCRIPTION}"}

        # display
        st.pydeck_chart(pyd.Deck(layers=[layer], initial_view_state=viewBoston, tooltip=dotOffense))

        # survey
        if st.checkbox("Optional Survey: Do you think Boston is safe? If so, please check the box."):
            st.markdown("Thank you for the response! :)")
