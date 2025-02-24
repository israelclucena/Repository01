it('should have initial properties set', () => {
  expect(component.loading).toBeTrue();
  expect(component.iframeElement).toBeUndefined();
  expect(component.height).toBeUndefined();
  expect(component.failedAttempts).toBe(0);
});
