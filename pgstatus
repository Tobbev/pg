import streamlit as st
import requests
import pandas as pd


username = st.secrets["username"]
password = st.secrets["password"]

token_url = 'https://api.zaptec.com/oauth/token'
data = {
    'grant_type': 'password',
    'username': username,
    'password': password
}

response = requests.post(token_url, data=data)

if response.status_code == 200:
    access_token = response.json().get('access_token')
    #st.write(f"Access Token: {access_token}")
else:
    st.write("Failed to obtain access token.")

headers = {
    'Authorization': f'Bearer {access_token}'
}

auth = 'Bearer ' + access_token

# Set page width
st.set_page_config(layout="wide")
st.title("Persikogatans Laddinfrastruktur")

def refresh():
    loading_text = st.empty()
    loading_text.text("Please wait. Probing API for data...")
    chargersList = getAllChargersIDs()
    loading_text.text(len(chargersList))
    
    chargerinfoDict = getChargersInfo(chargersList, loading_text)
    
    
    ChargersStates = getChargersStates(chargersList)
    
    headertext = ['Name', 'InstallationName', 'IsOnline', 'Signal Strength','Operating Mode', 'DeviceID', 'CircuitId', 'Active (Zaptec)', 'Pin', 'HasSessions', 'SignedMeterValueKwh', 'ID']
    table_data = [headertext]  # Initialize table data with header

    for charger in chargerinfoDict:
        params = [
            chargerinfoDict[charger]['Name'],
            chargerinfoDict[charger]['InstallationName'],
            chargerinfoDict[charger]['IsOnline'],
            ChargersStates[charger]['ConnectionStrength'],
            chargerinfoDict[charger]['OperatingMode'],
            chargerinfoDict[charger]['DeviceId'],
            chargerinfoDict[charger]['CircuitId'],
            chargerinfoDict[charger]['Active'],
            chargerinfoDict[charger]['Pin'],
            chargerinfoDict[charger]['HasSessions'],
            chargerinfoDict[charger]['SignedMeterValueKwh'],
            charger
        ]
        table_data.append(params)
    
    df = pd.DataFrame(table_data[1:], columns=table_data[0])
    total_rows = len(df)

    height_multiplier = 1.5
    table_height = int(25 * total_rows * height_multiplier)
          
    def highlight_red(val):
        return "background-color: red; color: white" if float(val) < -75 else ""

    # Apply the style to the "Signal Strength" column
    styled_df = df.style.applymap(highlight_red, subset=["Signal Strength"])

    # Display the DataFrame as a table
    st.dataframe(styled_df, height=table_height)

def getAllChargersIDs():
    url = 'https://api.zaptec.com/api/chargers'
    headers = {'accept': 'application/json', 'Authorization': auth}
    params = {'InstallationType': '0', 'ReturnIdNameOnly': 'true', 'SortDescending': 'true', 'IncludeDisabled': 'true'}

    response = requests.get(url, headers=headers, params=params)

    chargersIdList = []
    if response.status_code == 200:
        response_data = response.json()
        for item in response_data['Data']:
            chargersIdList.append(item['Id'])
    else:
        st.write('API request failed with status code:', response.status_code)
    
    return chargersIdList

def getChargersInfo(chargersIdList, loading_text):
    operatingModes = {0: 'Unknown', 1: 'Disconnected', 2: 'Connected_Requesting', 3: 'Connected_Charging', 5: 'Connected_Finished'}
    headers = {'accept': 'application/json', 'Authorization': auth}
    params = {'InstallationType': '0', 'ReturnIdNameOnly': 'true', 'SortDescending': 'true', 'IncludeDisabled': 'true'}

    
    

    chargers_processed = 0
    chargerDict = {}
    for charger_id in chargersIdList:
        url = f"https://api.zaptec.com/api/chargers/{charger_id}"
        
        
        total_chargers = len(chargersIdList)
        # Loop through each charger to probe for charger states
        
        chargers_processed += 1

        # Update the loading text with the progress
        loading_text.text(f"Probing charger {chargers_processed} of {total_chargers}...")
        
        response = requests.get(url, headers=headers)

        if response.status_code == 200:
            name = response.json()['Name']
            prefix, number = name.split(' ')
            formatted_number = number.zfill(3)
            formatted_name = f"{prefix} {formatted_number}"
            
            chargerDict[response.json()['Id']] = {
                'Name': formatted_name,
                'IsOnline': response.json()['IsOnline'],
                'OperatingMode': operatingModes.get(response.json().get('OperatingMode')),
                'DeviceId': response.json()['DeviceId'],
                'CircuitId': response.json()['CircuitId'],
                'Active': response.json()['Active'],
                'Pin': response.json()['Pin'],
                'HasSessions': response.json()['HasSessions'],
                'SignedMeterValueKwh': response.json()['SignedMeterValueKwh'],
                'InstallationName': response.json()['InstallationName'],
                'InstallationId': response.json()['InstallationId']
            }
        else:
            st.write(f"Failed to retrieve information for Charger ID: {charger_id}")
            st.write('API request failed with status code:', response.status_code)
            st.write()
    
    return chargerDict

def getChargersStates(chargersIdList):
    headers = {
        'accept': 'application/json',
        'Authorization': auth
    }
    params = {
        'InstallationType': '0',
        'ReturnIdNameOnly': 'true',
        'SortDescending': 'true',
        'IncludeDisabled': 'true'
    }
    chargerStatesDict = {}
    for charger_id in chargersIdList:
        url = f"https://api.zaptec.com/api/chargers/{charger_id}/state"
        response = requests.get(url, headers=headers)
        chargerStatesDict[charger_id] = {}
        
        if response.status_code == 200:
            #print(response.json())
            for state in response.json():
                if state['StateId'] == 809:
                    chargerStatesDict[charger_id]['ConnectionStrength'] = state['ValueAsString']
                    break

    return chargerStatesDict

refresh()
