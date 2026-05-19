import React, { useState } from 'react';
import { StyleSheet, Text, View, ScrollView, TouchableOpacity, SafeAreaView, TextInput, ActivityIndicator, Alert } from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

// Shared App Configuration / Mock Data Engine
const GLOBAL_EXCHANGE_RATE = 11.00; // Simulated free-market rate BOB/USDC
const mockUserData = {
  firstName: "Alejandro",
  usdcBalance: 1250.00,
  cardLastFour: "4832",
  cardExpiry: "12/29",
  transactions: [
    { id: '1', title: 'Remittance from Remote Co.', type: 'credit', amount: 450.00 },
    { id: '2', title: 'Netflix Subscription', type: 'debit', amount: 10.99 },
    { id: '3', title: 'QR Payment: Supermercado Fidalga', type: 'debit', amount: 12.50 },
  ]
};

const Tab = createBottomTabNavigator();

// ==========================================
// 1. DASHBOARD SCREEN COMPONENT
// ==========================================
function DashboardScreen({ navigation }: any) {
  const [showCardDetails, setShowCardDetails] = useState(false);
  const bobBalance = mockUserData.usdcBalance * GLOBAL_EXCHANGE_RATE;

  return (
    <SafeAreaView style={styles.container}>
      <ScrollView contentContainerStyle={styles.scrollContent}>
        
        {/* Header */}
        <View style={styles.header}>
          <Text style={styles.welcomeText}>Hola, {mockUserData.firstName} 👋</Text>
          <View style={styles.bellIcon}><Text style={{color: '#FFF'}}>🔔</Text></View>
        </View>

        {/* Balance Area */}
        <View style={styles.balanceContainer}>
          <Text style={styles.balanceLabel}>Tu balance disponible</Text>
          <Text style={styles.usdcText}>${mockUserData.usdcBalance.toFixed(2)} USDC</Text>
          <Text style={styles.bobText}>≈ {bobBalance.toLocaleString('es-BO')} BOB</Text>
        </View>

        {/* Quick Action Navigation Buttons */}
        <View style={styles.actionRow}>
          <TouchableOpacity style={styles.actionButton} onPress={() => navigation.navigate('Depositar')}>
            <Text style={styles.actionButtonText}>➕ Depósito</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.actionButton} onPress={() => Alert.alert("Enviar", "Módulo P2P en desarrollo")}>
            <Text style={styles.actionButtonText}>💸 Enviar</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.actionButton} onPress={() => setShowCardDetails(!showCardDetails)}>
            <Text style={styles.actionButtonText}>💳 Tarjeta</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.actionButton} onPress={() => navigation.navigate('Escanear QR')}>
            <Text style={styles.actionButtonText}>📱 Escanear</Text>
          </TouchableOpacity>
        </View>

        {/* Swipeable Virtual Card Mockup */}
        <Text style={styles.sectionTitle}>Tu Tarjeta Digital Global</Text>
        <View style={styles.creditCard}>
          <View style={styles.cardHeader}>
            <Text style={styles.cardBrand}>VISA</Text>
            <Text style={styles.cardType}>Prepaid</Text>
          </View>
          <Text style={styles.cardNumber}>
            {showCardDetails ? `4532 8192 0034 ${mockUserData.cardLastFour}` : `••••  ••••  ••••  ${mockUserData.cardLastFour}`}
          </Text>
          <View style={styles.cardFooter}>
            <Text style={styles.cardExpiry}>EXP: {mockUserData.cardExpiry}</Text>
            <TouchableOpacity onPress={() => setShowCardDetails(!showCardDetails)}>
              <Text style={styles.toggleDetailsText}>{showCardDetails ? "Ocultar" : "Ver Datos"}</Text>
            </TouchableOpacity>
          </View>
        </View>

        {/* Unified Transaction History Ledger */}
        <Text style={styles.sectionTitle}>Actividad Reciente</Text>
        {mockUserData.transactions.map((tx) => (
          <View key={tx.id} style={styles.txRow}>
            <View>
              <Text style={styles.txTitle}>{tx.title}</Text>
              <Text style={styles.txType}>{tx.type === 'credit' ? 'Ingreso' : 'Gasto Global'}</Text>
            </View>
            <Text style={[styles.txAmount, tx.type === 'credit' ? styles.creditText : styles.debitText]}>
              {tx.type === 'credit' ? '+' : '-'} ${tx.amount.toFixed(2)}
            </Text>
          </View>
        ))}

      </ScrollView>
    </SafeAreaView>
  );
}

// ==========================================
// 2. DEPOSIT SCREEN COMPONENT (Fiat & Cross-Border Forks)
// ==========================================
function DepositScreen() {
  const [activeTab, setActiveTab] = useState<'local' | 'global'>('local');
  const [bobAmount, setBobAmount] = useState('');
  const [showQR, setShowQR] = useState(false);

  const calculatedUSDC = bobAmount ? (parseFloat(bobAmount) / GLOBAL_EXCHANGE_RATE).toFixed(2) : '0.00';

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.tabContainer}>
        <TouchableOpacity style={[styles.tab, activeTab === 'local' && styles.activeTab]} onPress={() => { setActiveTab('local'); setShowQR(false); }}>
          <Text style={[styles.tabText, activeTab === 'local' && styles.activeTabText]}>QR Boliviano (BOB)</Text>
        </TouchableOpacity>
        <TouchableOpacity style={[styles.tab, activeTab === 'global' && styles.activeTab]} onPress={() => setActiveTab('global')}>
          <Text style={[styles.tabText, activeTab === 'global' && styles.activeTabText]}>Cuenta EE.UU. (USD)</Text>
        </TouchableOpacity>
      </View>

      {activeTab === 'local' ? (
        <View style={styles.content}>
          <Text style={styles.label}>¿Cuánto deseas depositar en BOB?</Text>
          <View style={styles.inputContainer}>
            <TextInput style={styles.input} placeholder="0.00" placeholderTextColor="#64748B" keyboardType="numeric" value={bobAmount} onChangeText={(text) => { setBobAmount(text); setShowQR(false); }} />
            <Text style={styles.inputCurrency}>BOB</Text>
          </View>
          <Text style={styles.exchangeHint}>Tipo de cambio paralelo: 1 USDC = {GLOBAL_EXCHANGE_RATE.toFixed(2)} BOB</Text>

          <View style={styles.conversionBox}>
            <Text style={styles.conversionLabel}>Recibirás en tu billetera:</Text>
            <Text style={styles.conversionValue}>${calculatedUSDC} USDC</Text>
          </View>

          {!showQR ? (
            <TouchableOpacity style={[styles.primaryButton, !bobAmount && styles.disabledButton]} disabled={!bobAmount} onPress={() => setShowQR(true)}>
              <Text style={styles.buttonText}>Generar QR de Pago</Text>
            </TouchableOpacity>
          ) : (
            <View style={styles.qrContainer}>
              <View style={styles.mockQR}>
                <Text style={{color: '#000', fontWeight: '700'}}>QR INTERBANCARIO</Text>
                <Text style={{color: '#2563EB', fontWeight: '600', marginTop: 10}}>{bobAmount} BOB</Text>
              </View>
              <Text style={styles.qrInstructions}>Toma un screenshot y págalo mediante Simple QR desde tu banco local (BNB, Mercantil, Unión, etc.). Tu saldo se acreditará inmediatamente.</Text>
            </View>
          )}
        </View>
      ) : (
        <View style={styles.content}>
          <Text style={styles.sectionDescription}>Usa tus credenciales bancarias estadounidenses personalizadas para facturar a clientes internacionales o retirar plataformas globales como Deel o PayPal.</Text>
          <View style={styles.bankCard}>
            <View style={styles.bankRow}><Text style={styles.bankLabel}>Banco Receptor</Text><Text style={styles.bankValue}>Evolve Bank & Trust</Text></View>
            <View style={styles.bankRow}><Text style={styles.bankLabel}>Beneficiario</Text><Text style={styles.bankValue}>Alejandro Mendoza</Text></View>
            <View style={styles.bankRow}><Text style={styles.bankLabel}>Número de Ruta (ACH/ABA)</Text><Text style={styles.copyValue}>021000021 📋</Text></View>
            <View style={styles.bankRow}><Text style={styles.bankLabel}>Número de Cuenta</Text><Text style={styles.copyValue}>123456789012 📋</Text></View>
            <View style={styles.bankRow}><Text style={styles.bankLabel}>Tipo</Text><Text style={styles.bankValue}>Checking / Corriente</Text></View>
          </View>
        </View>
      )}
    </SafeAreaView>
  );
}

// ==========================================
// 3. SCAN QR SCREEN COMPONENT (Local Cash-Out Simulator)
// ==========================================
function ScanQRScreen({ navigation }: any) {
  const [scanState, setScanState] = useState<'scanning' | 'confirming' | 'processing' | 'success'>('scanning');
  const invoiceBob = 165.00;
  const invoiceUsdc = (invoiceBob / GLOBAL_EXCHANGE_RATE).toFixed(2);

  return (
    <SafeAreaView style={styles.container}>
      {scanState === 'scanning' && (
        <View style={styles.fullFlex}>
          <Text style={styles.instructions}>Encuadra el código QR del comercio boliviano para pagar usando tus fondos estables</Text>
          <View style={styles.viewFinder}>
            <Text style={{color: '#64748B', fontSize: 12}}>Buscando QR Simple...</Text>
          </View>
          <TouchableOpacity style={styles.simulateButton} onPress={() => setScanState('confirming')}>
            <Text style={styles.simulateButtonText}>⚡ Simular Escaneo en Tienda</Text>
          </TouchableOpacity>
        </View>
      )}

      {scanState === 'confirming' && (
        <View style={styles.invoiceContainer}>
          <Text style={styles.sheetTitle}>Autorizar Gasto Local</Text>
          <View style={styles.invoiceCard}>
            <Text style={styles.bankLabel}>Establecimiento</Text>
            <Text style={styles.merchantName}>Supermercados Fidalga</Text>
            <View style={{height: 1, backgroundColor: '#334155', marginVertical: 15}} />
            <View style={styles.amountRow}><Text style={{color: '#94A3B8'}}>Monto Facturado:</Text><Text style={{color: '#FFF', fontWeight: '600'}}>{invoiceBob.toFixed(2)} BOB</Text></View>
            <View style={styles.amountRow}><Text style={{color: '#94A3B8'}}>Costo en Crypto:</Text><Text style={{color: '#10B981', fontWeight: '700'}}>${invoiceUsdc} USDC</Text></View>
          </View>
          <TouchableOpacity style={styles.payButton} onPress={() => { setScanState('processing'); setTimeout(() => setScanState('success'), 2000); }}>
            <Text style={styles.buttonText}>Confirmar y Pagar</Text>
          </TouchableOpacity>
          <TouchableOpacity style={{alignItems: 'center', marginTop: 10}} onPress={() => setScanState('scanning')}>
            <Text style={{color: '#64748B'}}>Cancelar</Text>
          </TouchableOpacity>
        </View>
      )}

      {scanState === 'processing' && (
        <View style={styles.centerFlex}>
          <ActivityIndicator size="large" color="#2563EB" />
          <Text style={{color: '#FFF', marginTop: 20, fontWeight: '600'}}>Ejecutando Permuta e Intercambio...</Text>
        </View>
      )}

      {scanState === 'success' && (
        <View style={styles.centerFlex}>
          <Text style={{fontSize: 50, marginBottom: 10}}>✅</Text>
          <Text style={styles.successTitle}>¡Liquidación Exitosa!</Text>
          <Text style={{color: '#94A3B8', textAlign: 'center', marginBottom: 30}}>Se transfirieron {invoiceBob} BOB instantáneamente al comercio.</Text>
          <TouchableOpacity style={styles.primaryButton} onPress={() => { setScanState('scanning'); navigation.navigate('Inicio'); }}>
            <Text style={[styles.buttonText, {paddingHorizontal: 30}]}>Volver al Inicio</Text>
          </TouchableOpacity>
        </View>
      )}
    </SafeAreaView>
  );
}

// ==========================================
// CENTRAL NAVIGATION TREE WRAPPER
// ==========================================
export default function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator
        screenOptions={{
          headerShown: false,
          tabBarStyle: { backgroundColor: '#1E293B', borderTopWidth: 0, height: 60, paddingBottom: 8 },
          tabBarActiveTintColor: '#2563EB',
          tabBarInactiveTintColor: '#64748B',
          tabBarLabelStyle: { fontWeight: '600', fontSize: 12 }
        }}
      >
        <Tab.Screen name="Inicio" component={DashboardScreen} />
        <Tab.Screen name="Depositar" component={DepositScreen} />
        <Tab.Screen name="Escanear QR" component={ScanQRScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}

// ==========================================
// UNIFIED STYLE LEDGER
// ==========================================
const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#0A0E1A' },
  scrollContent: { padding: 20 },
  header: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', marginBottom: 25 },
  welcomeText: { color: '#FFF', fontSize: 18, fontWeight: '600' },
  bellIcon: { backgroundColor: '#1E293B', padding: 10, borderRadius: 50 },
  balanceContainer: { marginBottom: 25 },
  balanceLabel: { color: '#94A3B8', fontSize: 14, marginBottom: 5 },
  usdcText: { color: '#FFF', fontSize: 36, fontWeight: '700' },
  bobText: { color: '#10B981', fontSize: 16, fontWeight: '500', marginTop: 2 },
  actionRow: { flexDirection: 'row', justifyContent: 'space-between', marginBottom: 30 },
  actionButton: { backgroundColor: '#2563EB', paddingVertical: 12, borderRadius: 12, width: '23%', alignItems: 'center' },
  actionButtonText: { color: '#FFF', fontSize: 11, fontWeight: '600' },
  sectionTitle: { color: '#FFF', fontSize: 17, fontWeight: '600', marginBottom: 15 },
  creditCard: { backgroundColor: '#1E40AF', padding: 20, borderRadius: 16, marginBottom: 30, height: 150, justifyContent: 'space-between' },
  cardHeader: { flexDirection: 'row', justifyContent: 'space-between' },
  cardBrand: { color: '#FFF', fontSize: 20, fontWeight: '800', fontStyle: 'italic' },
  cardType: { color: '#93C5FD', fontSize: 12 },
  cardNumber: { color: '#FFF', fontSize: 18, letterSpacing: 2 },
  cardFooter: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' },
  cardExpiry: { color: '#93C5FD', fontSize: 12 },
  toggleDetailsText: { color: '#FFF', fontSize: 12, fontWeight: '600', textDecorationLine: 'underline' },
  txRow: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', backgroundColor: '#1E293B', padding: 15, borderRadius: 12, marginBottom: 10 },
  txTitle: { color: '#FFF', fontSize: 14, fontWeight: '500' },
  txType: { color: '#64748B', fontSize: 12 },
  txAmount: { fontSize: 15, fontWeight: '600' },
  creditText: { color: '#10B981' },
  debitText: { color: '#EF4444' },
  tabContainer: { flexDirection: 'row', backgroundColor: '#1E293B', margin: 20, borderRadius: 8, padding: 4 },
  tab: { flex: 1, paddingVertical: 12, alignItems: 'center', borderRadius: 6 },
  activeTab: { backgroundColor: '#2563EB' },
  tabText: { color: '#94A3B8', fontWeight: '600', fontSize: 13 },
  activeTabText: { color: '#FFF' },
  content: { paddingHorizontal: 20 },
  label: { color: '#94A3B8', fontSize: 14, marginBottom: 10 },
  inputContainer: { flexDirection: 'row', alignItems: 'center', backgroundColor: '#1E293B', borderRadius: 12, paddingHorizontal: 15, marginBottom: 10 },
  input: { flex: 1, color: '#FFF', fontSize: 24, fontWeight: '700', paddingVertical: 12 },
  inputCurrency: { color: '#64748B', fontSize: 18, fontWeight: '600' },
  exchangeHint: { color: '#64748B', fontSize: 12, marginBottom: 20 },
  conversionBox: { backgroundColor: '#111827', borderLeftWidth: 4, borderLeftColor: '#10B981', padding: 15, borderRadius: 8, marginBottom: 25 },
  conversionLabel: { color: '#94A3B8', fontSize: 12, marginBottom: 5 },
  conversionValue: { color: '#10B981', fontSize: 22, fontWeight: '700' },
  primaryButton: { backgroundColor: '#2563EB', paddingVertical: 16, borderRadius: 12, alignItems: 'center' },
  disabledButton: { backgroundColor: '#1E293B', opacity: 0.5 },
  buttonText: { color: '#FFF', fontSize: 16, fontWeight: '600' },
  qrContainer: { alignItems: 'center', marginTop: 10 },
  mockQR: { width: 160, height: 160, backgroundColor: '#FFF', justifyContent: 'center', alignItems: 'center', borderRadius: 12, marginBottom: 15 },
  qrInstructions: { color: '#94A3B8', fontSize: 12, textAlign: 'center', lineHeight: 18 },
  sectionDescription: { color: '#94A3B8', fontSize: 13, lineHeight: 20, marginBottom: 20 },
  bankCard: { backgroundColor: '#1E293B', borderRadius: 16, padding: 20 },
  bankRow: { marginBottom: 12 },
  bankLabel: { color: '#64748B', fontSize: 11, marginBottom: 2 },
  bankValue: { color: '#FFF', fontSize: 14 },
  copyValue: { color: '#3B82F6', fontSize: 14, fontWeight: '600' },
  fullFlex: { flex: 1, justifyContent: 'space-between', alignItems: 'center', paddingVertical: 40 },
  centerFlex: { flex: 1, justifyContent: 'center', alignItems: 'center', padding: 30 },
  instructions: { color: '#94A3B8', fontSize: 14, textAlign: 'center', paddingHorizontal: 30 },
  viewFinder: { width: 220, height: 220, borderWidth: 2, borderColor: '#2563EB', borderStyle: 'dashed', justifyContent: 'center', alignItems: 'center', borderRadius: 20 },
  simulateButton: { backgroundColor: '#1E293B', paddingVertical: 14, paddingHorizontal: 25, borderRadius: 30, borderWidth: 1, borderColor: '#334155' },
  simulateButtonText: { color: '#3B82F6', fontWeight: '600', fontSize: 13 },
  invoiceContainer: { flex: 1, padding: 24, justifyContent: 'center' },
  sheetTitle: { color: '#FFF', fontSize: 20, fontWeight: '700', marginBottom: 20, textAlign: 'center' },
  invoiceCard: { backgroundColor: '#1E293B', padding: 20, borderRadius: 16, marginBottom: 25 },
  merchantName: { color: '#FFF', fontSize: 16, fontWeight: '600', marginTop: 2 },
  amountRow: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', marginVertical: 6 },
  payButton: { backgroundColor: '#10B981', paddingVertical: 16, borderRadius: 12, alignItems: 'center' },
  successTitle: { color: '#FFF', fontSize: 22, fontWeight: '700', marginBottom: 5 }
});
