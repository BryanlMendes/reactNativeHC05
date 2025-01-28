# reactNativeHC05
Bluetooth hc-05


import React, { useState, useEffect } from 'react';
import { View, Text, Button, ToastAndroid, PermissionsAndroid, Platform } from 'react-native';
import RNBluetoothClassic from 'react-native-bluetooth-classic';

const BluetoothApp = () => {
  const [bluetoothAvailable, setBluetoothAvailable] = useState(false);
  const [bluetoothEnabled, setBluetoothEnabled] = useState(false);
  const [discoveredDevices, setDiscoveredDevices] = useState([]);
  const [discovering, setDiscovering] = useState(false);
  const [connectedDevice, setConnectedDevice] = useState(null); // Estado para armazenar o dispositivo conectado

  useEffect(() => {
    checkBluetoothStatus();
    requestBluetoothPermissions();
    const onBluetoothEnabled = RNBluetoothClassic.onBluetoothEnabled(handleBluetoothEnabled);
    const onBluetoothDisabled = RNBluetoothClassic.onBluetoothDisabled(handleBluetoothDisabled);

    return () => {
      onBluetoothEnabled.remove();
      onBluetoothDisabled.remove();
    };
  }, []);

  const requestBluetoothPermissions = async () => {
    if (Platform.OS === 'android') {
      try {
        let granted;

        // Para Android 6 (API 23) até Android 11 (API 30)
        if (Platform.Version < 31) {
          granted = await PermissionsAndroid.requestMultiple([PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION]);
        } else {
          // Para Android 12 (API 31) e superior
          granted = await PermissionsAndroid.requestMultiple([
            PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
            PermissionsAndroid.PERMISSIONS.BLUETOOTH_CONNECT,
            PermissionsAndroid.PERMISSIONS.BLUETOOTH_SCAN,
          ]);
        }

        console.log('Permissões solicitadas:', granted);
        ToastAndroid.show('Solicitação de permissões em andamento...', ToastAndroid.SHORT);

        // Verificar se todas as permissões foram concedidas
        const allPermissionsGranted =
          granted['android.permission.ACCESS_FINE_LOCATION'] === PermissionsAndroid.RESULTS.GRANTED;

        if (Platform.Version >= 31) {
          // Para Android 12 e superior, verificar as permissões adicionais
          allPermissionsGranted &&
            granted['android.permission.BLUETOOTH_CONNECT'] === PermissionsAndroid.RESULTS.GRANTED &&
            granted['android.permission.BLUETOOTH_SCAN'] === PermissionsAndroid.RESULTS.GRANTED;
        }

        if (allPermissionsGranted) {
          console.log('Permissões de Bluetooth concedidas');
          ToastAndroid.show('Permissões de Bluetooth concedidas', ToastAndroid.SHORT);
        } else {
          console.log('Permissões de Bluetooth negadas');
          ToastAndroid.show('Permissões de Bluetooth negadas', ToastAndroid.SHORT);
        }
      } catch (err) {
        console.warn('Erro ao solicitar permissões:', err);
        ToastAndroid.show('Erro ao solicitar permissões', ToastAndroid.SHORT);
      }
    }
  };

  const checkBluetoothStatus = async () => {
    try {
      const available = await RNBluetoothClassic.isBluetoothAvailable();
      setBluetoothAvailable(available);

      const enabled = await RNBluetoothClassic.isBluetoothEnabled();
      setBluetoothEnabled(enabled);

      if (!enabled) {
        ToastAndroid.show('Por favor, ative o Bluetooth', ToastAndroid.SHORT);
      }
    } catch (err) {
      console.error('Bluetooth error:', err);
      ToastAndroid.show('Erro ao verificar Bluetooth', ToastAndroid.SHORT);
    }
  };

  const handleBluetoothEnabled = () => {
    setBluetoothEnabled(true);
  };

  const handleBluetoothDisabled = () => {
    setBluetoothEnabled(false);
  };

  const startDiscovery = async () => {
    if (!bluetoothEnabled) {
      ToastAndroid.show('Bluetooth não está habilitado', ToastAndroid.SHORT);
      return;
    }

    try {
      setDiscovering(true);
      const devices = await RNBluetoothClassic.startDiscovery();
      if (devices.length === 0) {
        ToastAndroid.show('Nenhum dispositivo encontrado', ToastAndroid.SHORT);
      } else {
        setDiscoveredDevices(devices);
        ToastAndroid.show(`Encontrado ${devices.length} dispositivos`, ToastAndroid.SHORT);
      }
    } catch (err) {
      console.error('Erro ao descobrir dispositivos:', err);
      ToastAndroid.show('Erro ao descobrir dispositivos', ToastAndroid.SHORT);
    } finally {
      setDiscovering(false);
    }
  };

  const connectToDevice = async (deviceAddress) => {
    try {
      const device = await RNBluetoothClassic.connectToDevice(deviceAddress);
      setConnectedDevice(device); // Armazena o dispositivo conectado
      ToastAndroid.show(`Conectado ao dispositivo ${device.name}`, ToastAndroid.SHORT);
    } catch (err) {
      console.error('Erro ao conectar ao dispositivo:', err);
      ToastAndroid.show('Erro ao conectar ao dispositivo', ToastAndroid.SHORT);
    }
  };

  const disconnectFromDevice = async () => {
    try {
      if (connectedDevice) {
        await RNBluetoothClassic.disconnectFromDevice(connectedDevice.address);
        setConnectedDevice(null); // Limpa o estado do dispositivo conectado
        ToastAndroid.show('Desconectado com sucesso', ToastAndroid.SHORT);
      }
    } catch (err) {
      console.error('Erro ao desconectar do dispositivo:', err);
      ToastAndroid.show('Erro ao desconectar do dispositivo', ToastAndroid.SHORT);
    }
  };

  const sendCommand = async (command) => {
    try {
      if (connectedDevice) {
        // Usando writeToDevice() para enviar o comando
        const success = await RNBluetoothClassic.writeToDevice(connectedDevice.address, command);
        
        if (success) {
          console.log(`Comando "${command}" enviado com sucesso`);
        } else {
          console.log(`Erro ao enviar comando: ${command}`);
        }
      }
    } catch (err) {
      console.error('Erro ao enviar comando:', err);
    }
  };

  const cancelDiscovery = async () => {
    try {
      const result = await RNBluetoothClassic.cancelDiscovery();
      ToastAndroid.show('Descoberta de dispositivos cancelada', ToastAndroid.SHORT);
    } catch (err) {
      console.error('Erro ao cancelar descoberta:', err);
      ToastAndroid.show('Erro ao cancelar descoberta', ToastAndroid.SHORT);
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Bluetooth Disponível: {bluetoothAvailable ? 'Sim' : 'Não'}</Text>
      <Text>Bluetooth Habilitado: {bluetoothEnabled ? 'Sim' : 'Não'}</Text>
      <View>
        <Button title="Acender LED" onPress={() => sendCommand('1')} />
        <Button title="Apagar LED" onPress={() => sendCommand('0')} />
      </View>
      {connectedDevice && (
        <>
          <Text>Conectado a: {connectedDevice.name}</Text>
          <Button title="Desconectar" onPress={disconnectFromDevice} />
        </>
      )}

      {!connectedDevice && (
        <>
          <Button
            title="Iniciar Descoberta"
            onPress={startDiscovery}
            disabled={discovering || !bluetoothEnabled}
          />
          {discovering && <Text>Descobrindo dispositivos...</Text>}
          <Button
            title="Cancelar Descoberta"
            onPress={cancelDiscovery}
            disabled={!discovering}
          />
          <View>
            {discoveredDevices.length > 0 && <Text>Dispositivos Descobertos:</Text>}
            {discoveredDevices.map((device) => (
              <View key={device.address}>
                <Text>{device.name}</Text>
                <Button title="Conectar" onPress={() => connectToDevice(device.address)} />
              </View>
            ))}
          </View>
        </>
      )}
    </View>
  );
};

export default BluetoothApp;
