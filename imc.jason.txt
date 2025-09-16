import React, { useState, useEffect } from "react";
import { View, Text, TextInput, TouchableOpacity, StyleSheet, FlatList, SafeAreaView, StatusBar, Switch } from "react-native";
import { ProgressBar } from 'react-native-paper';

export default function App() {
  const [weight, setWeight] = useState(0);
  const [goal, setGoal] = useState(0);
  const [consumed, setConsumed] = useState(0);
  const [log, setLog] = useState([]);
  const [lastDrinkTime, setLastDrinkTime] = useState(Date.now());
  const [timeSinceLastDrink, setTimeSinceLastDrink] = useState("0s");
  const [darkMode, setDarkMode] = useState(false);

  useEffect(() => {
    const calculatedGoal = Math.round(weight * 35); // 35ml por kg
    setGoal(calculatedGoal);
  }, [weight]);

  useEffect(() => {
    const interval = setInterval(() => {
      const seconds = Math.floor((Date.now() - lastDrinkTime) / 1000);
      const h = Math.floor(seconds / 3600);
      const m = Math.floor((seconds % 3600) / 60);
      const s = seconds % 60;
      setTimeSinceLastDrink(`${h}h ${m}m ${s}s`);
    }, 1000);
    return () => clearInterval(interval);
  }, [lastDrinkTime]);

  const addWater = (amount) => {
    const newTotal = consumed + amount;
    setConsumed(newTotal);
    setLastDrinkTime(Date.now());
    setLog((prev) => [...prev, { id: Date.now().toString(), amount }]);
  };

  const resetDay = () => {
    setConsumed(0);
    setLog([]);
    setLastDrinkTime(Date.now());
  };

  const progress = goal > 0 ? consumed / goal : 0;

  const getAvatar = () => {
    if (progress < 0.1) return "😱";
    if (progress < 0.3) return "😢";
    if (progress < 0.5) return "😒";
    if (progress < 0.7) return "🙂";
    if (progress < 0.9) return "😃";
    return "🥰";
  };

  const getProgressColor = () => {
    if (progress <= 1) return "#00796b";
    return "#ff9800"; // cor alaranjada se ultrapassou a meta
  };

  const theme = darkMode ? stylesDark : styles;

  return (
    <SafeAreaView style={theme.container}>
      <StatusBar barStyle={darkMode ? "light-content" : "dark-content"} backgroundColor={darkMode ? "#121212" : "#e0f7fa"} />
      <View style={theme.switchContainer}>
        <Text style={theme.label}>Modo escuro</Text>
        <Switch value={darkMode} onValueChange={setDarkMode} />
      </View>
      <Text style={theme.title}> Contador de Água 💧</Text>
      <Text style={theme.avatar}>{getAvatar()}</Text>
      <TextInput
        style={theme.input}
        keyboardType="numeric"
        placeholder="Digite seu peso (kg)"
        placeholderTextColor="#aaa"
        onChangeText={(text) => setWeight(parseFloat(text) || 0)}
      />
      <View style={theme.infoBox}>
        <Text style={theme.infoText}>Meta diária: <Text style={theme.highlight}>{goal} ml</Text></Text>
        <Text style={theme.infoText}>Consumido: <Text style={theme.highlight}>{consumed} ml</Text></Text>
        <Text style={theme.infoText}>Tempo desde o último copo: <Text style={theme.highlight}>{timeSinceLastDrink}</Text></Text>
        <ProgressBar progress={progress > 1 ? 1 : progress} color={getProgressColor()} style={theme.progressBar} />
        {progress > 1 && (
          <Text style={[theme.infoText, { color: '#ff9800', marginTop: 6 }]}>Você ultrapassou sua meta! 🎉</Text>
        )}
      </View>

      <View style={theme.buttonsContainer}>
        <TouchableOpacity style={theme.button} onPress={() => addWater(200)}>
          <Text style={theme.buttonText}>+200ml</Text>
        </TouchableOpacity>
        <TouchableOpacity style={theme.button} onPress={() => addWater(300)}>
          <Text style={theme.buttonText}>+300ml</Text>
        </TouchableOpacity>
        <TouchableOpacity style={[theme.button, theme.resetButton]} onPress={resetDay}>
          <Text style={theme.buttonText}>Resetar</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        data={log}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <Text style={theme.logItem}>🟦 +{item.amount}ml</Text>
        )}
      />
    </SafeAreaView>
  );
}

const baseStyles = {
  container: {
    flex: 1,
    padding: 20,
  },
  switchContainer: {
    flexDirection: "row",
    justifyContent: "flex-end",
    alignItems: "center",
  },
  label: {
    marginRight: 8,
    fontSize: 16,
  },
  avatar: {
    fontSize: 50,
    textAlign: "center",
    marginVertical: 10,
  },
  title: {
    fontSize: 30,
    fontWeight: "bold",
    marginBottom: 10,
    textAlign: "center",
  },
  input: {
    borderWidth: 1,
    padding: 12,
    marginVertical: 10,
    borderRadius: 12,
    fontSize: 16,
  },
  infoBox: {
    padding: 15,
    borderRadius: 10,
    marginBottom: 20,
  },
  infoText: {
    fontSize: 18,
    marginBottom: 4,
  },
  highlight: {
    fontWeight: "bold",
  },
  progressBar: {
    height: 10,
    borderRadius: 5,
    marginTop: 10,
  },
  buttonsContainer: {
    flexDirection: "row",
    justifyContent: "space-around",
    marginBottom: 20,
  },
  button: {
    paddingVertical: 12,
    paddingHorizontal: 18,
    borderRadius: 10,
  },
  resetButton: {},
  buttonText: {
    fontSize: 16,
    fontWeight: "bold",
    textAlign: "center",
  },
  logItem: {
    fontSize: 16,
    padding: 6,
  },
};

export const styles = StyleSheet.create({
  ...baseStyles,
  container: {
    ...baseStyles.container,
    backgroundColor: "#e0f7fa",
  },
  title: {
    ...baseStyles.title,
    color: "#006064",
  },
  input: {
    ...baseStyles.input,
    borderColor: "#004d40",
    backgroundColor: "#ffffff",
    color: "#333",
  },
  infoBox: {
    ...baseStyles.infoBox,
    backgroundColor: "#b2ebf2",
  },
  infoText: {
    ...baseStyles.infoText,
    color: "#004d40",
  },
  highlight: {
    ...baseStyles.highlight,
    color: "#006064",
  },
  progressBar: {
    ...baseStyles.progressBar,
    backgroundColor: "#e0f2f1",
  },
  button: {
    ...baseStyles.button,
    backgroundColor: "#00838f",
  },
  resetButton: {
    ...baseStyles.resetButton,
    backgroundColor: "#d32f2f",
  },
  buttonText: {
    ...baseStyles.buttonText,
    color: "#fff",
  },
  logItem: {
    ...baseStyles.logItem,
    color: "#00796b",
  },
});

export const stylesDark = StyleSheet.create({
  ...baseStyles,
  container: {
    ...baseStyles.container,
    backgroundColor: "#121212",
  },
  title: {
    ...baseStyles.title,
    color: "#ffffff",
  },
  input: {
    ...baseStyles.input,
    borderColor: "#666",
    backgroundColor: "#1e1e1e",
    color: "#fff",
  },
  infoBox: {
    ...baseStyles.infoBox,
    backgroundColor: "#1c1c1c",
  },
  infoText: {
    ...baseStyles.infoText,
    color: "#cccccc",
  },
  highlight: {
    ...baseStyles.highlight,
    color: "#90caf9",
  },
  progressBar: {
    ...baseStyles.progressBar,
    backgroundColor: "#333",
  },
  button: {
    ...baseStyles.button,
    backgroundColor: "#0277bd",
  },
  resetButton: {
    ...baseStyles.resetButton,
    backgroundColor: "#b71c1c",
  },
  buttonText: {
    ...baseStyles.buttonText,
    color: "#fff",
  },
  logItem: {
    ...baseStyles.logItem,
    color: "#4fc3f7",
  },
});


// Os estilos continuam os mesmos...